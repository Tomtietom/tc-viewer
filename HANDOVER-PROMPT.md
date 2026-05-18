# Handover-prompt voor de tc-viewer sessie

Plak onderstaande tekst (alles onder de regel) als prompt in de sessie die je voor `tc-viewer` hebt open staan. De prompt is zelf-bevattend.

---

In deze sessie staat het project `tc-viewer` open — een Trimble Connect Workspace-API extensie (vanilla JS, single-file `index.html` van 5205 regels).

We hebben in een aparte review-sessie het project tegen de officiële OpenAPI 3.0 spec van de Trimble Connect Core REST API gelegd. Het volledige review-rapport staat op:

- **Rapport**: `./REVIEW-vs-OpenAPI.md` (relatief vanuit project-root)
- **Spec ter referentie**: `../Trimble Connect API/openapi-spec` (YAML)

Voer alleen de **P1-fixes** uit het rapport uit. Twee stuks:

## Fix 1 — `pageSize=1000` overschrijdt spec-maximum (500)

**Locatie**: `index.html:1506`
**Spec**: `openapi-spec:267-274` definieert max `pageSize=500` voor v2.1 `/folders/{folderId}/items`.

Verander op regel 1506 `?pageSize=1000` naar `?pageSize=500`. Eenregelig.

## Fix 2 — v2.1 cursor leest verkeerde veldnaam, stopt na 1 page

**Locatie**: `index.html:1510`
**Spec**: response heeft `links.next.href` (volledige URL) — géén los `links.next.skipToken` veld.

De huidige code:
```javascript
skipToken = (resp && resp.links && resp.links.next && resp.links.next.skipToken)
         || (resp && resp.skipToken) || null;
```

Vervang door:
```javascript
var nextHref = resp && resp.links && resp.links.next && resp.links.next.href;
if (nextHref) {
  var m = /[?&]skipToken=([^&]+)/.exec(nextHref);
  skipToken = m ? decodeURIComponent(m[1]) : null;
} else {
  skipToken = null;
}
```

Belangrijk: behoud de bestaande breakvoorwaarde rond regel 1511. Empty items met aanwezige `links.next.href` is per spec mogelijk; alleen op `!nextHref` breken is robuuster maar niet strict nodig.

## Verificatie

1. **Sanity-check**: laad de extensie in een project met een folder die >500 items bevat (typisch een grote release-folder of model-folder). Vóór de fix stopte het na 1 page (max 1000 — maar mogelijk 400-error → v2.0 fallback). Na de fix moet de UI alle items tonen, gepaginate in batches van 500.
2. **Console**: bij correcte werking verschijnt `[TC] v2.1 items endpoint niet beschikbaar` **niet** voor folders waar het eerst wel verscheen.
3. **Netwerk-tab** in browser DevTools: controleer dat `/tc/api/folders/{id}/items?pageSize=500&skipToken=...` calls verschijnen voor grote folders.

## Niet in scope voor deze sessie

P2 en P3 bevindingen uit het rapport (NA-region alias, v2.0 fallback pagination) — laat staan. PSet API en At Fielt Hub vallen buiten de Core API spec.

Maak één commit per fix met duidelijke message. Push pas na bevestiging dat het werkt in een live project.
