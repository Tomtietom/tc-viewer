# Review tc-viewer vs Trimble Connect OpenAPI spec

**Spec**: `../Trimble Connect API/openapi-spec` (OpenAPI 3.0, v2.0/v2.1)
**Reviewed**: hoofdbestand `index.html` (5205 regels)
**Datum**: 2026-05-11

## Samenvatting

Goed gebouwde extensie. Het project doet al veel wat de skill voorschrijft (region discovery met cache, JWT exp-check, retry-logica, request-id logging, v2.1 met v2.0 fallback). Twee concrete bugs ten opzichte van de spec gevonden, plus enkele kleinere observaties.

## Bevindingen (gerangschikt)

### P1 — pageSize=1000 overschrijdt spec-maximum (500)

**Locatie**: `index.html:1506`

```javascript
var qp = '?pageSize=1000' + (skipToken ? '&skipToken=' + encodeURIComponent(skipToken) : '');
var resp = await tcFetchV21('/folders/' + folderId + '/items' + qp);
```

**Spec**: `openapi-spec:267-274` (v2.1 `/folders/{folderId}/items`):
> **Default:** 100, **Maximum:** 500, **Minimum:** 1
> *"If the `pageSize` parameter is greater than specified limit endpoint can support server will return error to help client avoid bug in pagination implementation."*

**Risico**: server kan 400 teruggeven → fallback naar v2.0 → grote folders inefficiënt of incompleet.

**Fix**: zet `pageSize` op `500`. Dezelfde correctie in `tcFetchV21` voor `/2.1/folders/by_path` wanneer die gebruikt wordt.

---

### P1 — v2.1 cursor-extractie leest verkeerde veldnaam

**Locatie**: `index.html:1510`

```javascript
skipToken = (resp && resp.links && resp.links.next && resp.links.next.skipToken)
         || (resp && resp.skipToken) || null;
```

**Spec**: `openapi-spec:147-151`, `345-349` en de top-level beschrijving (regel 51):
> *"The `links.next.href` property in the response contains the URL to the next page of results."*

De spec definieert alleen `links.next.href` (een volledige URL); geen los `skipToken` veld in de response. De code probeert `links.next.skipToken` te lezen — die bestaat per spec niet. Bij de eerste page wordt `skipToken` daarom `null` en stopt de loop. Voor folders met >1000 items pakt het project nu **alleen de eerste page** (en valt vervolgens niet eens terug op v2.0, want de v2.1 call slaagde technisch).

**Fix**: parse `resp.links.next.href` en extraheer de `skipToken` query-parameter, of volg `href` direct:

```javascript
var nextHref = resp && resp.links && resp.links.next && resp.links.next.href;
if (nextHref) {
  var m = /[?&]skipToken=([^&]+)/.exec(nextHref);
  skipToken = m ? decodeURIComponent(m[1]) : null;
} else { skipToken = null; }
```

**Aanvullend**: de breakvoorwaarde `if (!skipToken || !items.length) break;` is verder correct — empty items met aanwezige `links.next.href` is volgens spec mogelijk (regel 379-388 over /projects), dus alleen op `!nextHref` breaken zou robuuster zijn.

---

### P2 — `/folders/{folderId}/items` (v2.0) accepteert geen Range/pagination

**Locatie**: `index.html:1522` (v2.0 fallback)

**Spec**: `openapi-spec:3698-3734` — de v2.0 GET heeft alleen `folderId` (path) en `include` (query) parameters; response is een `200` array, geen `206`. Geen Range-header parameter gedefinieerd.

**Observatie**: de huidige code geeft geen Range mee en accepteert de volledige array — dat is correct. Maar als de v2.0 endpoint zelf intern paginated (TC's algemene "Paginated Responses" sectie zegt "All collections/lists support a Range header"), dan kan de fallback bij hele grote folders alsnog afgekapt zijn. Niet kritisch zolang de meeste folders via v2.1 worden opgehaald.

**Geen actie nodig**, behalve P1-fix toepassen zodat v2.1 daadwerkelijk werkt.

---

### P3 — Region match: 'asia' alias ontbreekt 'asia pacific'

**Locatie**: `index.html:1064`

```javascript
'asia': ['asia','ap','as','ap-southeast','ap-1']
```

**Spec**: `openapi-spec:4425, 4461` geven `location: "northAmerica"` als response-voorbeeld. Live `/regions` kan strings als `"Asia Pacific"`, `"AP2"`, `"Asia"` teruggeven.

**Risico**: laag — `targetAliases.some(function(a) { return ... id.indexOf(a) >= 0; })` doet substring-match, dus `"asia pacific"` matcht `"asia"`. Maar `northAmerica` (camelCase) krijgt `loc = "northamerica"` (lowercased zonder spatie) en zou met de huidige aliases-key `'north america'` (mét spatie) **niet matchen**. Test met een NA-project of de region discovery dan wel pakt.

**Fix**: normaliseer ook camelCase → spaced ("northAmerica" → "north america") voor de lookup, of voeg `'northamerica'` als directe alias-key toe.

---

### Informatief — Wel goed gedaan

- **Region discovery via `/regions`** met 24u-cache (regel 1031-1075) — voldoet aan spec en skill-best-practice.
- **`/projects/{id}/users` met `Range: items=X-Y`** header (regel 4150-4151) — spec ondersteunt dit (openapi-spec:5184-5238, response is 206). Loop stopt zodra eigen email gevonden — efficiënt.
- **`/releases?projectId=`** en **`/releases/{id}/files`** (regel 1247, 1255) — passen op spec regels 5392 en 5549.
- **Token refresh** met 60s margin (regel 970-1008) — best-practice.
- **Idempotent retry** met exponential backoff + Retry-After (regel 1120-1180) — voldoet aan spec-impliciete 429/5xx-conventie.
- **Request-ID extraction** uit `tc-request-id`, `X-Azure-Ref`, `x-amzn-RequestId` (regel 1107) — exact wat trimble-mcp-server ook doet.
- **v2.1 → v2.0 fallback** op error (regel 1514) — robuust patroon, maar wordt door P1#2 nu te vroeg getriggerd of helemaal niet.

### Buiten scope

- **PSet proxy via `tc-proxy.tom-da0.workers.dev`** (regel 631-633) — niet in deze OpenAPI spec. Eigen API, niet beoordeeld.
- **At Fielt Extension Hub** — eigen toegangscontrole, geen Trimble endpoint.

## Aanbevolen vervolgactie

1. Apply P1 #1 (pageSize=500) — eenregelig.
2. Apply P1 #2 (parse `links.next.href`) — kleine refactor van de v2.1 pagination-helper.
3. Test met een project dat een folder met >1000 items bevat om #2 te valideren.
4. P3 alias-fix als ooit een NA-project moet worden ondersteund.
