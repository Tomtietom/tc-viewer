# Backlog — tc-viewer

Open punten en ideeën voor volgende sessies. Geprioriteerd op impact.

Laatst bijgewerkt: 2026-04-25.

---

## ✅ Recent afgerond (referentie)

Opgenomen in v3.2 (april 2026):
- **Volledige metadata** via achtergrond-scan op per-bestand PSet endpoint (commit `999e71d`)
- Voortgangs-indicator + verfijnde banner-tekst (commit `5c7edc5`)
- 404-spam in browser-console weggewerkt via tc-proxy update (commit `e169834` + Cloudflare deploy)

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

## 🐛 Bugs (open)

### Filter-bug (open sinds 2026-04-20)

Gebruiker meldde: filters werken niet goed. Symptomen onbekend — Martin's mail noemde geen specifiek filter-probleem.

**Bij volgende sessie vragen:**
- Welk filter precies? (Bestandstype-chips, kolom-dropdowns, release, metadata-aanwezigheid, mappen, zoekbox)
- In welk project en met welke filter-combinatie?
- Screenshot/console-output

**Verdachten:**
- Werkt `resetFilters()` volledig na recente wijzigingen (includedExts, extChips)?
- Combinatie metadata-filter + extensie-filter + map-selectie consistent?
- Filter-dropdown voor dynamisch-EDITABLE velden werkt?

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
