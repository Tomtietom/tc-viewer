# Backlog — tc-viewer

Open punten en ideeën voor volgende sessies. Geprioriteerd op impact.

Laatst bijgewerkt: 2026-05-06.

---

## ✅ Recent afgerond (referentie)

Opgenomen in v3.3 (mei 2026):

**Bugfixes (Martin's lijst van 30 april + vervolg):**
- **Zoekfunctie werkt direct na laden** — was pas actief na handmatige profielwissel. Oorzaak: race tussen async hub-config-load en eerste render. Nu altijd herrender via `finally`-block in `loadProjectColConfig`, plus `_zoekKeys`-fallback op `ALL_COLS` als `getCols()` te vroeg leeg is. Zoekbox-listener via `oninput`-attribuut op het input-element zelf — geen JS-bind-timing meer.
- **Bestandstype-chips en groepering werken direct** — zelfde oorzaak als zoek, dezelfde fix.
- **3D-bestand → 3D viewer met model** — was naar `/detailviewer?fileId=X&versionId=` (lege version). Nu officiële deeplink `/viewer/3d?modelId=X` (uit docs.3d.connect.trimble.com).
- **PDF / 2D-bestand → 2D viewer met preview** — `/viewer/2D?id=X&version=Y` met versionId nu gevuld vanuit `_versionId`.
- **"Open in TC" link in tabel + detail-paneel** — opent nu daadwerkelijk het bestand i.p.v. lege verkenner.
- **Map-link (↗) in detail-paneel** — gebruikt nu `/data/folder/{folderId}` (TC's eigen URL-patroon, bevestigd via handmatige navigatie). Oude `/files/{folderId}` toonde lege verkenner.
- **PDF-preview triggert geen download meer** — MIME geforceerd op `application/pdf` op de blob; iframe-versie verwijderd (Chrome blokkeerde blob: in geneste iframe). Knop "PDF openen" opent als preview in nieuw tabblad.
- **Metadata-edits blijven behouden na hard refresh** — kritieke regressie. TC's PSet API is eventually consistent én inconsistent tussen endpoints (`/psets`, `/versions`, changeset-response retourneren elk verschillende velden). `saveCell` bouwt changeset-props nu uit een UNION van álle bronnen (`doc[]` + deep-cache + edits-buffer + `/versions`) — geen single source of truth meer. Plus localStorage-buffer (`editBuf:{pid}`, 30 min TTL) die hard refresh overleeft en automatisch opruimt zodra TC zichzelf bijgewerkt heeft.

**Onder de motorkap:**
- Cache-busting meta-tag (`<meta name="build">`) bumpen forceert browser/SW-revalidatie van de extensie-iframe.
- `_psetDeepCacheSet` wordt nu ook gevuld vanuit bulk-match (was alleen vanuit deep-scan en saveCell) — geeft latere saves meer historische data om de UNION mee te bouwen.
- Sanity check via `/versions` na elke save: diagnostische log die TC's inconsistentie zichtbaar maakt zonder data te overschrijven.

Opgenomen in v3.2 (april 2026):
- **Volledige metadata** via achtergrond-scan op per-bestand PSet endpoint (commit `999e71d`)
- Voortgangs-indicator + verfijnde banner-tekst (commit `5c7edc5`)
- 404-spam in browser-console weggewerkt via tc-proxy update (commit `e169834` + Cloudflare deploy)
- **Projecten zonder PSet-schema** netjes ondersteund (commits `263bd18`, `cf05446`, `d843f20`, `c389a2e`):
  - Filter/edit-UI verborgen + duidelijke uitleg-banner
  - Hardcoded VW-fallback uitgezet (geen fictieve velden meer)
  - Tabel toont alleen relevante technische kolommen
  - Mappenboom toont nu álle mappen, ook lege/diepe paden
  - Groeperen op mappenpad werkt direct
  - Footer-count `X mappen` klopt met TC's eigen verkenner

Opgenomen in v3.1 (april 2026):
- Auto-aanvullen bij vrije-tekstvelden
- Klikbare mappenlocatie in detailpaneel
- Auto-refresh banner bij nieuwe/gewijzigde documenten
- Versie-keuze bij release toevoegen, delen, ZIP-download
- Versie-historie in detailpaneel
- Diagnose-knop rechts onderin
- Soepelere kolombreedtes (sleep-zone, dubbelklik, reset)
- Download vanuit release volgt mappenstructuur

---

## 🛡️ Trimble best-practice future-proofing (open)

Geïdentificeerd na skill-pass (2026-04-28) tegen de officiële Trimble specs. Geen acuut defect; pak op zodra een aanleiding ontstaat.

1. **HTTP 206 + `Content-Range` support** — canoniek paginatie-patroon volgens spec. PSet-endpoint negeert Range-headers nu (zie comment regel 1471), maar als Trimble dit aanpast: in `tcFetch` bij `r.status === 206` parse `Content-Range: items 0-99/2611` → return `{items, total}`. Caller kan dan op `total` stoppen i.p.v. cursor.
2. **SRI-hash + lokale fallback voor workspace-api UMD** — bescherming tegen CDN-uitval. `<script src=…/index.js integrity="sha384-..." onerror="loadLocalFallback()">`. Pas relevant als CDN-storingen ooit issues geven.
3. **Concurrency cap herzien bij 429-spikes** — `_DEEP_CONCURRENCY=10` is OK; bij echte rate-limit-pieken evt. dynamische scaling naar 5. Backoff-helper (toegevoegd 2026-04-28) vangt 429 al netjes op.
4. **`tcAPI.project.getCurrentProject` fallback verwijderen** — defensieve fallback toegevoegd 2026-04-28 in regel 908. Verwijderen wanneer alle live TC-instances bevestigd zijn op `getProject`.
5. **PSet API region-discovery** — PSet zit niet in `/regions` response; `PSET_REGIONS` blijft hardcoded. Als Trimble ooit een `/regions`-equivalent voor PSet API publiceert, dan deze ook dynamisch maken.

---

## 🐛 Bugs (open)

Geen bekende open bugs. Filter-bug van 20 april (filters/zoek/groepering werkten pas na profielwissel) is opgelost in v3.3.

---

## ✨ Features (open, prioritair)

### Release-tab eigen meerwaarde geven

**Probleem:** De release-tab is nu een afgeslankte kopie van de doclijst. Martin's vraag: wat is de meerwaarde?

**Visie:** Release-tab moet de "operationele" plek worden voor release-workflow:

| Kan vanuit doclijst | Kan vanuit release-tab (huidig) | Moet kunnen vanuit release-tab |
|---|---|---|
| ✅ Filteren | ❌ | ✅ |
| ✅ Losse docs selecteren | ❌ | ✅ |
| ✅ Kolom-sortering | ❌ | ✅ |
| ✅ ZIP download met mappenstructuur | ✅ (nu gefixt) | ✅ |
| — | — | ✅ Filter op release-status (open/gesloten/ontvanger/verzender) |

**Implementatie schets:**
- Release-detail pagina krijgt zelfde tabel-component als doclijst (hergebruik `renderT()` met `filterByRelease` optie)
- Filter-balk bovenaan release-detail: status (open/gesloten/concept/archief), ontvanger, verzender
- Per-document checkboxes + bulk-acties (download geselecteerde subset, deel geselecteerde subset)
- Als dit klaar is: eventueel de release-keuzelijst in doclijst-zijbalk weghalen → duidelijker verschil tussen de twee tabs

**Keuze om te maken:** Hergebruik van de doclijst-tabel in release-view vs. een eigen compactere variant.

### Filter op release-status

In de release-overzichtspagina (`pgRel`) filters toevoegen:

- **Status**: open / gesloten / concept
- **Rol**: ontvanger / verzender
- **Datum-range**: aanmaakdatum / deadline

Plek: bovenaan `pgRel`, naast de huidige lijst. Styling consistent met doclijst-filterbar.

---

## 🎨 UI / UX verbeteringen (open)

### Indicator-positie en zichtbaarheid (uit v3.2-werk)

**Huidig:** Pulsende pill rechts onderin (`Metadata laden — X van Y`). Werkt, maar kan verfijnd:
- Bij grote tabellen scrollt de pill mee onderaan, soms uit zicht
- Tom kan overwegen 'm sticky te plakken aan de onderrand van de viewport

### Banner-tekst en banner-styling

**Huidig:** Banner toont nu 3 verschillende teksten (tijdens scan / lage dekking / legacy). Werkt, maar:
- Visueel zelfde gele tint voor zowel info als waarschuwing
- Kan baat hebben bij een soft "info"-variant (lichtblauw) tijdens scan i.p.v. waarschuwingsgeel
- Banner-positie is bovenin tabel — bij smalle vensters duwt 'ie content naar beneden

### Re-render tijdens scan optimaliseren

**Huidig:** Elke 50 voltooide calls → volledige `renderT()`. Bij grote tabellen (Vestzicht 2611 docs) kost dit zichtbaar tijd.
**Beter:** Per-rij DOM-update voor de specifieke docs die metadata kregen. Alleen nodig als users last melden.

### Mobiel (Safari) ondersteuning

**Status:** Bekende beperking — TC extensies werken niet op mobile Safari (apart geheugenitem). Niet in extensie op te lossen.

### Collapsible sidebar-secties

Bij projecten met veel filters + kolommen scrolt de zijbalk lang. Idee:
- Filters / Bestandstype / Weergave / Mappen elk in een uitklapbare sectie
- Onthouden welke open zijn per project (sessionStorage)

Alleen doen als de huidige stijl-consolidatie onvoldoende rust heeft gebracht.

### Header-dropdown voor secundaire acties

ZIP, CSV, Releases, Delen kunnen onder een ⋮-menu in de header. Maakt primaire acties prominenter. Alleen als gebruikers melden dat de huidige top-bar te druk oogt.

### Diagnose-rapport: visualisatie i.p.v. tekst

**Huidig:** Knop kopieert pure tekst naar klembord.
**Beter (klein):** Modal met de tekst getoond + één-klik kopiëren. Voor support handiger om mee te lezen.

---

## 💡 Ideeën / later

### Hub: extension_config voor andere extensie-voorkeuren

Per project kunnen we via de hub opslaan: filter-defaults, groepering-default, default-kolomprofiel-per-rol. Hub-database heeft al de velden, alleen UI rond.

### Token-refresh debug-knop

Voor support-scenario's: een knop in Diagnose-modal die `_tcTokenExp = Date.now() - 1000` zet en een actie uitvoert, zodat we live kunnen zien of token-refresh werkt zonder te hoeven wachten op verloop.

### TC-support ticket: 100-instance limiet bulk-PSet

Onze deep-scan bouwt om het 100-limiet heen, maar 750 calls voor één project blijft een workaround. Ticket bij Trimble met verzoek om:
- Verhoging of opheffing 100-limiet op `/libs/{lib}/psets`
- Of: bulk-query endpoint (`POST /psets/_query` of vergelijkbaar)

Bij positief antwoord: deep-scan-loop vervangen door 1 call.

### Batch-endpoint in tc-proxy

Mochten we toch in proxy zitten: `POST /pset/eu/batch` dat een array frns accepteert en alleen gevonden items retourneert. Voor 750 bestanden: 1 request i.p.v. 750. Niet kritiek nu (8 sec scan is snel), maar mooi voor schaalbaarheid.

### v2.1 items endpoint onderzoeken

Geeft 400 op `/folders/{id}/items?pageSize=1000`. Apart onderzoek waard — kan zijn dat er een verplichte query-parameter mist of dat we Range-header moeten gebruiken in plaats van pageSize.

---

## 🔒 Niet meer relevant (verwijderd uit deze backlog)

Voor de zekerheid genoteerd voor wie deze file later leest:
- ~~Bug: Download vanuit release platte mapstructuur~~ → opgelost in v3.1
- ~~Feature: Auto-aanvullen vrije-tekstvelden~~ → opgelost in v3.1
- ~~Feature: Volledige metadata bij grote projecten~~ → opgelost in v3.2
- ~~Bug: Metadata mist bij grote projecten (paginatie)~~ → opgelost in v3.2 deep-scan
- ~~UI: 404-spam in browser-console~~ → opgelost in v3.2 via tc-proxy update
- ~~Bug: Zoek/filter/groepering werkten pas na profielwissel~~ → opgelost in v3.3 (race condition tussen async hub-config en eerste render)
- ~~Bug: Klik op bestand opent leeg tabblad (3D / PDF / link / bestandsnaam)~~ → opgelost in v3.3 (correcte viewer-deeplinks)
- ~~Bug: Map-link (↗) in detail toonde lege verkenner~~ → opgelost in v3.3 (`/data/folder/{folderId}` URL-patroon)
- ~~Bug: PDF-preview triggert download i.p.v. preview~~ → opgelost in v3.3 (MIME-fix + iframe-removal i.v.m. Chrome blob:-blok)
- ~~Bug: Metadata-edits raken kwijt na hard refresh~~ → opgelost in v3.3 (UNION-merge in saveCell + localStorage-buffer)
