---
name: dom-inspect
description: "Navigeer naar een testpagina en documenteer de relevante DOM-selectors voor een A/B test. Gebruik deze skill wanneer je de DOM van een pagina moet inspecteren om selectors en structuur te begrijpen.

<example>
user: \"Inspecteer de DOM van de checkout pagina op wehkamp.nl\"
assistant: Navigeert naar de pagina en documenteert DOM-selectors
</example>

<example>
user: \"Welke selectors moet ik gebruiken voor het orderoverzicht?\"
assistant: Voert dom-inspect uit op de relevante pagina
</example>"
---

# DOM Inspectie voor A/B Test

Gebruik deze skill om de DOM van een pagina te inspecteren en relevante selectors te documenteren.

## Stappen

### 1. Navigeer naar de pagina

Gebruik `mcp__Claude_in_Chrome__navigate` om naar de test-URL te navigeren.

Als de pagina inloggen of een specifieke state vereist (bijv. winkelwagen gevuld, checkout), noteer dit dan. Vraag Nino eventueel om de pagina handmatig te openen als je er niet zelf bij kunt.

### 2. Lees de paginabron

Gebruik `mcp__Claude_in_Chrome__read_page` of `mcp__Claude_in_Chrome__javascript_tool` om:
- De HTML-structuur van het relevante gebied te lezen
- De classnames van de te wijzigen elementen te achterhalen

### 3. Identificeer selectors

Zoek selectors voor de elementen die wijzigen in de test:
- Let op CSS Modules → class names met hash-suffixen → gebruik altijd `[class*="StableClassName"]`
- Controleer of React de DOM kan herschrijven (SPA-gedrag) → MutationObserver noodzakelijk
- Zoek stabiele parent-elementen voor de MutationObserver

Gebruik `mcp__Claude_in_Chrome__javascript_tool` om selectors te testen:
```javascript
document.querySelector('[class*="StableClassName"]')
```

### 4. Documenteer de DOM-structuur

Maak een geordend overzicht van de relevante elementen:

| # | Element | Selector | Opmerkingen |
|---|---------|----------|-------------|
| 0 | Naam element | `[class*="..."]` | bijv. direct child van sticky section |
| 1 | ... | ... | ... |

Noteer ook:
- **Overkoepelende containers** (parent-elementen voor MutationObserver)
- **Framework-gedrag** (React SPA, Vue, etc.)
- **Bijzonderheden** (lazy loading, dynamisch gegenereerde content, A/B tool interference)

### 5. Output

Schrijf de bevindingen weg naar de experiment CLAUDE.md onder de sectie:
- `## DOM Selectors`

En voeg toe aan de klant-CLAUDE.md als het nieuwe, herbruikbare selectors zijn die ook relevant zijn voor toekomstige tests.
