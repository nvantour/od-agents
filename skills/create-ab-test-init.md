---
name: ab-test-init
description: "Initialiseer een nieuw A/B test experiment volledig: analyseer het Figma design, maak de mappenstructuur aan, maak een CLAUDE.md op basis van het template, en inspecteer de DOM van de testpagina. Gebruik deze skill wanneer Nino start met een nieuwe A/B test.

<example>
user: \"Maak een nieuwe test aan op basis van deze Figma URL: https://figma.com/...\"
assistant: Voert ab-test-init uit om het experiment volledig op te zetten
</example>

<example>
user: \"Start een nieuwe A/B test voor Wehkamp: [Figma URL]\"
assistant: Voert ab-test-init uit
</example>"
---

# A/B Test Initialisatie

Gebruik deze skill om een nieuw experiment volledig op te zetten. Voer de stappen in volgorde uit.

## Stap 1 — Analyseer het Figma design

Voer de `figma-analyse` skill uit. Die extraheert het experiment ID, testnaam, devices en varianten, en documenteert de visuele verschillen per variant en device.

## Stap 2 — Bepaal de mapnaam

Combineer het experiment ID en de testnaam tot een mapnaam in kebab-case:

**Formaat**: `[ID]-[beschrijving-in-kebab-case]`

Voorbeelden:
- `OD25 | Checkout | Show product images in order overview` → `OD25-show-product-images-in-order-overview`
- `OD52 | PDP | Multibuy in buying block` → `OD52-multibuy-buying-block`

## Stap 3 — Maak de mappenstructuur aan

Maak de experimentmap aan in `klanten/[klantnaam]/experimenten/[mapnaam]/`:

```
[mapnaam]/
├── CLAUDE.md           # Experiment documentatie (zie stap 4)
├── variation-a.js      # Control-build (leeg placeholder)
├── variation-b.js      # Variant B-build (leeg placeholder)
├── tampermonkey-a.js   # Control voor lokaal testen (leeg placeholder)
└── tampermonkey-b.js   # Variant B voor lokaal testen (leeg placeholder)
```

Voeg extra bestanden toe indien van toepassing:
- Variant C aanwezig → ook `variation-c.js` en `tampermonkey-c.js`

## Stap 4 — Maak de experiment CLAUDE.md aan

Gebruik het template op `.claude/templates/experiment-claude-template.md` als basis.

Vul in wat je al weet vanuit de Figma analyse:
- Test info (ID, naam, device, tool)
- Beschrijving per variant (A = control, B/C = variaties)
- Visuele verschillen tabel

Laat nog open:
- `## DOM Selectors` (wordt ingevuld in stap 5)
- `## Code-architectuur` (wordt ingevuld bij ontwikkeling)
- Hypothese (vraag aan Nino als die er nog niet is, of laat leeg)

## Stap 5 — Inspecteer de DOM

Voer de `dom-inspect` skill uit. Die navigeert naar de test-URL, inspecteert de relevante elementen en vult de `## DOM Selectors` sectie in de experiment CLAUDE.md in.

> De test-URL staat in de klant-CLAUDE.md. Als de pagina een speciale state vereist (inloggen, gevulde winkelwagen, etc.) — vraag Nino om de pagina handmatig te openen.

## Stap 6 — Bevestig aan Nino

Geef een korte samenvatting:
- ✅ Mapnaam + locatie
- ✅ Aanwezig: varianten + devices
- ✅ Visuele verschillen (kort samengevat)
- ✅ DOM selectors gedocumenteerd
- ⏭️ Volgende stap: code schrijven via `ab-test-development`

## Naamgevingsregels

- Map: altijd in **kebab-case**, **lowercase**
- Prefix = experiment ID uit de Figma paginanaam (bijv. `OD`, `ID`)
- Spaties → koppeltekens, speciale tekens weglaten
