---
name: figma-analyse
description: "Analyseer een Figma design voor een A/B test. Extraheer automatisch varianten, devices en visuele verschillen. Gebruik deze skill wanneer een Figma URL beschikbaar is bij het opzetten van een A/B test.

<example>
user: \"Hier is de Figma URL: https://www.figma.com/design/...\"
assistant: Voert figma-analyse uit om varianten, devices en verschillen te documenteren
</example>

<example>
user: \"Analyseer het Figma design voor OD58\"
assistant: Voert figma-analyse uit op de opgegeven Figma URL
</example>"
---

# Figma Analyse voor A/B Test

Gebruik deze skill om een Figma design te analyseren en de relevante informatie te extraheren voor een A/B test.

## Stappen

### 1. Lees het Figma design

Gebruik de `figma-remote-mcp` MCP om het design te lezen:
- Gebruik `get_metadata` om de paginastructuur en -namen te zien
- Gebruik `get_design_context` met de node-id voor elk relevant frame
- Gebruik `get_screenshot` voor een visuele vergelijking van varianten

### 2. Extraheer structuurinformatie uit de paginanaam

De paginanaam volgt het formaat: `[ID] | [Onderdeel] | [Naam van de test] | [Device] | [Variant]`

Voorbeeld: `OD25 | Checkout | Show product images in order overview | DT | Variant B`

Extraheer hieruit:
- **Experiment ID**: bijv. `OD25`
- **Naam**: de beschrijvende testnaam (wordt de mapnaam)
- **Devices**: DT = Desktop, M = Mobile — noteer welke combinaties aanwezig zijn
- **Varianten**: A = control, B / C / D = variaties

### 3. Identificeer alle varianten en devices

Maak een overzicht:

| Variant | Device | Aanwezig |
|---------|--------|----------|
| A | DT | ✓ / ✗ |
| A | M | ✓ / ✗ |
| B | DT | ✓ / ✗ |
| B | M | ✓ / ✗ |

Aandachtspunten:
- Variant A = altijd de **huidige situatie** (control/bestaand design)
- Variant B, C etc. = nieuwe variatie(s)
- Code wordt gemaakt voor ALLE devices die aanwezig zijn in het design

### 4. Analyseer de visuele verschillen

Vergelijk elke variant (B, C...) met de control (A) per device:
- Wat is er visueel **toegevoegd**?
- Wat is er **verwijderd** of **verborgen**?
- Wat is er **verplaatst**?
- Wat is er **aangepast** in stijl, tekst of layout?

Wees zo specifiek mogelijk: afmetingen, kleuren, posities, tekstwijzigingen.

**Bij afbeeldingen in het design — noteer altijd:**
- De afmeting in het design (bijv. 80×80px)
- De viewport-breedte van het design frame (bijv. 375px voor mobile)
- Dit is nodig om de juiste `clamp()` waarden te berekenen bij de implementatie

### 5. Output

Documenteer de bevindingen in de experiment CLAUDE.md onder de secties:
- `## Varianten` — beschrijving per variant
- `## Visuele verschillen` — gestructureerd overzicht van de wijzigingen

Geef ook aan welke bestanden aangemaakt moeten worden op basis van het aantal varianten en devices.
