# Backlog — tc-viewer

Zaken die opgepakt moeten worden in een volgende sessie, niet urgent genoeg voor directe fix.

## Filter-bug onderzoeken (2026-04-20)

Gebruiker meldt: **het filteren werkt niet goed**. Symptomen nog onbekend — bij volgende sessie vragen:

- Welk filter precies? (Bestandstype-chips, kolom-dropdowns, release, metadata-aanwezigheid, mappen, zoekbox)
- In welk project en met welke filter-combinatie?
- Screenshot/console-output helpt

Mogelijke verdachten om te controleren:
- Werkt `resetFilters()` volledig na de recente wijzigingen (includedExts, extChips)?
- Is de combinatie metadata-filter + extensie-filter + map-selectie consistent?
- Worden chip-waarden correct opgeslagen bij profile-switch?
- Sommige velden die nu pas dynamisch EDITABLE zijn — filter-dropdown voor die velden werkt?

## Ideeën / later

- Hub: extension_config ook voor andere extensie-voorkeuren per project (filter-defaults, groepering-default)
- Token-refresh ook expliciet test-knop voor debugging (handmatig `_tcTokenExp = Date.now() - 1000` en actie uitvoeren)
