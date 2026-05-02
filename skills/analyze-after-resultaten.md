---
name: analyse
description: "Gebruik deze skill wanneer Nino testresultaten wil analyseren of een test-analyse wil maken. Volgt een vast format met resultaten, segmentatie, conclusie en learnings.\n\n<example>\nuser: \"De test op Wehkamp is klaar, hier zijn de resultaten\"\nassistant: Volgt automatisch het analyse-format\n</example>\n\n<example>\nuser: \"Analyseer de resultaten van OD52\"\nassistant: Volgt automatisch het analyse-format\n</example>"
---

# Test Analyse

Gebruik dit format bij het analyseren van testresultaten.

## Stappen

### 1. Verzamel test informatie
Vraag naar of zoek op:
- **Test ID**: [IDxx]
- **Klant**: [klantnaam]
- **Pagina/flow**: [waar draaide de test]
- **Looptijd**: [startdatum] t/m [einddatum]
- **Tool**: [VWO/Convert/Optimizely]

### 2. Herhaal de hypothese
Zoek de originele hypothese op in de experiment-map of vraag Nino ernaar.

### 3. Resultaten in tabel
| Variant | Bezoekers | Conversies | Conversie % | Uplift | Significantie |
|---------|-----------|------------|-------------|--------|---------------|
| Control | | | | — | — |
| Variant B | | | | | |

### 4. Segmentatie
Vraag altijd naar resultaten per segment:
- **Device**: desktop vs. mobiel
- **Nieuw vs. terugkerend**
- **Overige relevante segmenten** (verkeersbron, land, etc.)

### 5. Conclusie
- **Winnaar**: [Control / Variant X / Geen verschil]
- **Statistische significantie bereikt**: [Ja / Nee]
- **Aanbeveling**: [Implementeren / Itereren / Stoppen]

### 6. Learnings
Eindig altijd met:
- Wat hebben we geleerd, ongeacht het resultaat?
- Wat is de volgende logische test?

## Belangrijk
- Waarschuw als de test te kort heeft gelopen (< 1 business cycle / < 7 dagen)
- Waarschuw als het aantal bezoekers te laag is voor betrouwbare resultaten
- Kijk altijd naar segmenten — een overall neutraal resultaat kan per segment significant zijn
