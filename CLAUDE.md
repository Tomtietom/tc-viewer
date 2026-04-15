# At Fielt Trimble Connect extensie — Documentenlijst

Deze extensie (interne naam: tc-viewer) toont een gestructureerde documentenlijst met metadata, filters en export-mogelijkheden voor een Trimble Connect project. Geïntegreerd met de At Fielt Extension Hub voor toegangscontrole en usage-tracking.

## Werkwijze voor Claude Code sessies

Wanneer je deze extensie bewerkt:

- **Bewaar de hub-integratie blokken** tussen `╔══ AT FIELT EXTENSION HUB — NIET VERWIJDEREN ══╗` en `╔══ EINDE AT FIELT HUB BLOK ══╗` ongewijzigd, tenzij je het hub-contract aanpast.
- **De hub-script tag** bovenaan `<head>` niet verwijderen:
  ```html
  <script src="https://atfielt-extension-hub.tom-da0.workers.dev/client.js"></script>
  ```
- **De `[HUB] logEvent` calls** niet verwijderen — je mag wel het `data` object uitbreiden met extensie-specifieke tracking-velden.
- **Volledige integratie-referentie**: https://github.com/Tomtietom/atfielt-extension-hub/blob/main/HUB-INTEGRATION.md

## Publiceren

Na wijzigingen in `index.html`:
```bash
git add index.html
git commit -m "<korte beschrijving wat je wijzigde>"
git push
```

Wacht 1-3 min op GitHub Pages rebuild, dan verifieer:
1. Open extensie in Trimble Connect
2. Console (F12) → zoek `[Hub] Toegang verleend`
3. https://atfielt-extension-hub.tom-da0.workers.dev/admin → Events tab → nieuw `load` event verschijnt binnen 30s

## Deze extensie

- **Slug in hub-database**: `documentenlijst`
- **GitHub repo**: Tomtietom/tc-viewer
- **Manifest URL**: https://tomtietom.github.io/tc-viewer/
- **Hub URL**: https://atfielt-extension-hub.tom-da0.workers.dev
- **Admin dashboard**: https://atfielt-extension-hub.tom-da0.workers.dev/admin
