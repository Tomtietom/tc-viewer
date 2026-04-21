# Backlog — tc-viewer

Zaken die opgepakt moeten worden in een volgende sessie. Geprioriteerd op impact.

---

## Feedback Martin (2026-04-21)

Uit mail met bugmeldingen + verbeterpunten. Metadata-bug is al gefixt (paginatie + robuuste VID-match in commit `b476eb0`). Rest staat hieronder.

---

### 🐛 Bug — Download vanuit release geeft andere mapstructuur

**Symptoom:** Als je een release downloadt via de release-tab, komt de output als een "platte" / verkorte mappenstructuur binnen. Terwijl hetzelfde bestand vanuit de doclijst wél in de volledige structuur wordt gezipt. Gebruiker verwacht gelijk gedrag.

**Te onderzoeken:**
- Waar zit de verschillende ZIP-logica: `downloadZIP()` (doclijst) vs `downloadRelease()` (release-tab)
- `file_location` moet bij release-bestanden ook de volledige TC-mappenpad bevatten
- Release-bestanden hebben mogelijk alleen `name` en missen het `parentId`-pad — dan moeten we het pad aanvullen vanuit `docs[]` via VID-match

**Acceptatie:** Download vanuit release = identieke ZIP als download vanuit doclijst met dezelfde bestanden.

---

### ✨ Feature — Release-tab eigen meerwaarde geven

**Probleem:** De release-tab is nu een afgeslankte kopie van de doclijst. Martin's vraag: wat is de meerwaarde?

**Visie:** Release-tab moet de "operationele" plek worden voor release-workflow:

| Kan vanuit doclijst | Kan vanuit release-tab (huidig) | Moet kunnen vanuit release-tab |
|---|---|---|
| ✅ Filteren | ❌ | ✅ |
| ✅ Losse docs selecteren | ❌ | ✅ |
| ✅ Kolom-sortering | ❌ | ✅ |
| ✅ ZIP download met mappenstructuur | ❌ (platte ZIP) | ✅ |
| — | — | ✅ Filter op release-status (open/gesloten/ontvanger/verzender) |

**Implementatie schets:**
- Release-detail pagina krijgt zelfde tabel-component als doclijst (hergebruik `renderT()` met `filterByRelease` optie)
- Filter-balk bovenaan release-detail: status (open/gesloten/concept/archief), ontvanger, verzender
- Per-document checkboxes + bulk-acties (download geselecteerde subset, deel geselecteerde subset)
- Als dit klaar is: eventueel de release-keuzelijst in doclijst-zijbalk weghalen → duidelijker verschil tussen de twee tabs

**Keuze om te maken:**
- Hergebruik van de doclijst-tabel in release-view vs. een eigen compactere variant

---

### ✨ Feature — Filter op release-status

In de release-overzichtspagina (pgRel) filters toevoegen:

- **Status**: open / gesloten / concept
- **Rol**: ontvanger / verzender
- **Datum-range**: aanmaakdatum / deadline

Plek: bovenaan `pgRel`, naast de huidige lijst. Styling consistent met doclijst-filterbar.

---

### ✨ Feature — Auto-aanvullen bij vrije-tekstvelden

**Martin:** *"Norman doelt erop dat als je begint te typen je een voorstel krijgt van wat al in die velden staat. Niet een standaard vooringevulde lijst — dit beperkt de flexibiliteit."*

**Scope:** Elk metadata-veld dat handmatig invoer heeft (niet enum). Concreet in ieder geval: `bedrijf`, `titel`, `bouwdeel`, `bouwnummer`, `document_nummer`, `document_soort`, `tags`.

**Implementatie:**
- Bij edit-mode: vervang `<input type="text">` door input met autocomplete-lijst (HTML5 `<datalist>` met unieke waarden uit andere docs voor dat veld)
- Werkt ook in de bulk-edit input
- Geen hardcoded lijst — alleen voorstellen uit wat er al in het project staat

**Voordeel:** Consistentie in data (minder varianten van "VolkerWessels" / "Volker Wessels" / "VW"), maar gebruiker blijft vrij om af te wijken.

---

### 🐛 Filter-bug (open sinds 2026-04-20)

Gebruiker meldde: **filters werken niet goed**. Symptomen nog onbekend — Martin's mail gaf geen specifiek filter-probleem aan, dus dit blijft een apart item.

Bij volgende sessie vragen:
- Welk filter precies? (Bestandstype-chips, kolom-dropdowns, release, metadata-aanwezigheid, mappen, zoekbox)
- In welk project en met welke filter-combinatie?
- Screenshot/console-output helpt

Verdachten:
- Werkt `resetFilters()` volledig na de recente wijzigingen (includedExts, extChips)?
- Combinatie metadata-filter + extensie-filter + map-selectie consistent?
- Filter-dropdown voor dynamisch-EDITABLE velden werkt?

---

## Ideeën / later

- Hub: extension_config ook voor andere extensie-voorkeuren per project (filter-defaults, groepering-default)
- Token-refresh: handmatige test-knop voor debugging (`_tcTokenExp = Date.now() - 1000` en actie uitvoeren)
- UI-simplicatie stap 2: collapsible sidebar-secties + header-dropdown voor secundaire acties (ZIP, CSV). Alleen doen als de stijl-consolidatie onvoldoende rust heeft gebracht.
