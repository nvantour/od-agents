---
name: ab-test-development
description: "Volg deze richtlijnen wanneer je een A/B test ontwikkelt (JavaScript variatiecode). Wordt automatisch getriggerd wanneer Nino vraagt om een test te bouwen, variatiecode te schrijven, of een experiment te ontwikkelen.

<example>
user: \"Bouw een test die de CTA-tekst wijzigt op de PDP\"
assistant: Volgt automatisch de ab-test-development richtlijnen
</example>

<example>
user: \"Schrijf de variatiecode voor OD53\"
assistant: Volgt automatisch de ab-test-development richtlijnen
</example>"
---

# A/B Test Development Richtlijnen

Volg deze richtlijnen bij het ontwikkelen van elke A/B test.

## VERPLICHT: altijd minimaal twee varianten bouwen

Elke test levert **altijd** minstens twee bestanden op:

1. **Control (A)** — géén DOM-wijzigingen, géén CSS. Alleen het INIT event + eventuele gedeelde click-tracking. Zorgt dat de controlegroep meetbaar is.
2. **Variant (B)** — alle DOM/CSS-wijzigingen + INIT event + variant-specifieke tracking.

(Bij een 3-variant test komt er een Variant C bij.)

Nooit alleen een variant bouwen — zonder control mist de controlegroep tracking en is de test niet meetbaar.

## Bestandsstructuur per experiment

Elk experiment staat in `klanten/[naam]/experimenten/[IDxx-beschrijving]/`.

**Standaard (nieuwe test):**
```
[IDxx-beschrijving]/
├── CLAUDE.md
├── variation-b.js        # Productiebuild voor de testingtool (VWO/Convert/Optimizely)
├── variation-a.js        # Control-build voor de testingtool
├── tampermonkey-b.js     # Tampermonkey versie voor lokaal testen
└── tampermonkey-a.js     # Control tampermonkey (indien nodig)
```

**Working-version structuur (voor iteratieve experimenten, bijv. Keesing):**
```
[IDxx-beschrijving]/
├── [subcategorie]/
│   ├── current-version/
│   │   ├── convert.js
│   │   └── tampermonkey.js
│   └── working-version/
│       ├── convert.js
│       └── tampermonkey.js
```

Check altijd de klant-CLAUDE.md voor de exacte mapstructuur van een experiment.

## Naamgeving

- Map: `[IDxx]-[korte-beschrijving]` in kebab-case lowercase (bijv. `ID19-payment-method`)
- Prefix = projectcode + volgnummer uit testingtool (`ID` voor Keesing, `OD` voor Wehkamp, `AB` voor Mantel, etc.)
- Bestandsnamen per tool: `variation-b.js`, `tampermonkey-b.js`, `convert.js`
- 3-variant test: `variation-b.js`, `variation-c.js`, `tampermonkey-b.js`, `tampermonkey-c.js`

## Code-structuur: IIFE

Alle variatiecode is een **IIFE** (`immediately invoked function expression`). Gebruik altijd `var`, niet `const`/`let` — testingtools draaien soms in omgevingen zonder volledige ES6 ondersteuning.

```javascript
(function () {
  'use strict';

  // ... code hier

})();
```

## Tracking — boilerplate (verplicht bij Convert/Optimizely)

Bij **Convert, Optimizely en Kameleoon** zit er geen eigen tracking in de tool — die moet je zelf inbouwen. Gebruik altijd deze standaard boilerplate inline in elk bestand:

```javascript
(function () {
  'use strict';

  // CONFIG
  var config = {
    category: 'AB-test',
    testId: 'ID20',      // experiment ID
    devices: 'DTM',      // DTM = Desktop, Tablet, Mobile — pas aan als nodig
    variant: 'B',        // 'A' voor control, 'B'/'C' voor varianten
  };

  // Debug logging (alleen zichtbaar met OD-cookie)
  var getCookie = function (name) {
    var v = document.cookie.match('(^|;) ?' + name + '=([^;]*)(;|$)');
    return v ? v : null;
  };

  var log = function () {
    var msg = Array.prototype.slice.call(arguments);
    if (getCookie('OD')) {
      console.log('%c' + config.testId + ': %o', 'margin: 15px; color: pink; font-size: 21px;', msg);
    }
  };

  // Event tracking naar dataLayer
  var oldEvent = '';
  var sendEvent = function (extra, nonInteraction, label) {
    extra = extra ? ' : ' + extra : '';
    nonInteraction = nonInteraction || false;
    label = label || '';
    var dataString = config.testId + config.variant + ' - ' + config.devices + extra;
    if (oldEvent === dataString) return;  // deduplicatie
    oldEvent = dataString;
    setTimeout(function () { oldEvent = ''; }, 250);
    log(dataString, label);
    if (!window.dataLayer) window.dataLayer = [];
    window.dataLayer.push({
      'event': 'ab-test',
      'eventCategory': config.category,
      'eventAction': dataString,
      'eventLabel': label,
      'eventNonInteraction': nonInteraction,
    });
  };

  // Body class voor CSS scoping
  var addBodyClass = function () {
    if (document.body) {
      document.body.classList.add(config.testId + config.variant);
    } else {
      setTimeout(addBodyClass, 50);
    }
  };
  addBodyClass();

  // INIT event — ALTIJD als eerste versturen in alle varianten
  sendEvent('Init', true);

  // ... rest van de code (DOM, styles, click-tracking)

})();
```

**Control (variant A):** zelfde boilerplate, géén DOM-wijzigingen, géén CSS — alleen `sendEvent('Init', true)` en eventuele gedeelde click-tracking.

## Tracking bij VWO

VWO heeft eigen tracking-mechanismen. Variatiecode voor VWO bevat vaak **geen eigen sendEvent boilerplate** — de tool registreert het INIT event zelf. Check de klant-CLAUDE.md voor hoe tracking is ingericht bij die klant.

## Tampermonkey-bestand

Het tampermonkey-bestand is identiek aan de productiebuild, maar met een UserScript-header:

```javascript
// ==UserScript==
// @name         ID20 - Testnaam - VARIANT B
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  Korte beschrijving
// @match        https://www.website.nl/*
// @grant        none
// @run-at       document-start
// ==/UserScript==

(function () {
  'use strict';
  // ... zelfde code als de productiebuild
})();
```

## INIT event — verplicht in alle varianten

`sendEvent('Init', true)` **moet altijd worden verstuurd**, ook in de control. Zonder dit event is de controlegroep niet meetbaar in de data.

## DOM wachten — polling met setInterval

Gebruik `setInterval` om te wachten tot DOM-elementen aanwezig zijn. Stop de polling zodra de initialisatie slaagt, en zet een timeout als veiligheidsmaatregel:

```javascript
var pollInterval = setInterval(function () {
  if (init()) {
    clearInterval(pollInterval);
  }
}, 300);

setTimeout(function () {
  clearInterval(pollInterval);
}, 15000);
```

De `init()` functie geeft `true` terug als alle benodigde elementen gevonden zijn en de code is uitgevoerd.

## MutationObserver voor SPA-navigatie

Bij React/Vue sites kan de DOM herschreven worden na navigatie. Gebruik een MutationObserver om de initialisatie opnieuw te triggeren als de variant verdwijnt:

```javascript
var observer = new MutationObserver(function () {
  if (!document.getElementById('id20-mijn-element')) {
    init();
  }
});

observer.observe(document.body, { childList: true, subtree: true });
```

## CSS isolatie

Voeg een body-klasse toe (`[testId][variant]`, bijv. `ID20B`) voor CSS scoping:

```css
.ID20B .element { ... }
```

**Alle** CSS class names, element IDs, style-tag IDs en keyframe-namen moeten geprefixed zijn met `[testId]-` (lowercase):

```javascript
// Element IDs
document.createElement('div').id = 'id20-mijn-element';
// Style tag
style.id = 'id20-styles';
```
```css
/* CSS classes */
.id20-mijn-class { ... }
/* Keyframes */
@keyframes id20-spin { ... }
```

Voorkomt naming-collisions met bestaande site-classes. Check altijd of er geen generieke prefix in de code zit.

## CSS injecteren — altijd via `<style>` tag

Gebruik **nooit inline styles** voor CSS die responsive gedrag nodig heeft. Injecteer altijd via een `<style>` element met een uniek ID als guard:

```javascript
function injectStyles() {
  if (document.getElementById('id20-styles')) return;
  var style = document.createElement('style');
  style.id = 'id20-styles';
  style.textContent = '.id20-element { ... }';
  document.head.appendChild(style);
}
```

Voordelen: ondersteunt media queries en `clamp()`, makkelijk te overschrijven, voorkomt duplicaten bij re-renders.

## Klant-specifieke richtlijnen

Lees ALTIJD eerst de klant-CLAUDE.md in `klanten/[naam]/CLAUDE.md` voor platform-specifieke constraints (selectors, framework-gedrag, CSS modules, trackingaanpak, etc.)

## Afbeeldingen

### Altijd `object-fit: contain`
Gebruik **nooit** `object-fit: cover` voor productafbeeldingen — dit snijdt het product af. Gebruik altijd `contain` met een lichte achtergrond:

```css
.id20-product-img {
  object-fit: contain;
  background: #f5f5f5;
}
```

### Responsieve afbeeldingsgrootte — gebruik `clamp()`
Gebruik **nooit** vaste px-waarden voor afbeeldingsafmetingen. Gebruik `clamp()`:

```css
.id20-product-img {
  width: clamp(minPx, Xvw, maxPx);
  height: clamp(minPx, Xvw, maxPx);
}
```

- `maxPx` = de Figma-specificatie (bijv. 80px)
- `Xvw` = zo dat `Xvw` ≈ `maxPx` bij de Figma-viewport-breedte (bijv. 21vw bij 375px ≈ 80px)
- `minPx` = ondergrens voor kleine schermen (bijv. 56px)

### Breakpoints controleren
Zoek vóór het schrijven van code het breakpoint van het element op:

```javascript
Array.from(document.styleSheets).forEach(function (sheet) {
  try {
    Array.from(sheet.cssRules).forEach(function (rule) {
      if (rule.type === CSSRule.MEDIA_RULE && rule.cssText.includes('NaamVanElement'))
        console.log(rule.conditionText, rule.cssText.substring(0, 200));
    });
  } catch(e) {}
});
```

## Mobile-first details

### Tekst die op exact N regels moet passen
Vaste height + line-clamp, anders duwen langere woorden de layout uit het ritme:

```css
.id20-title {
  font-size: 11px;
  line-height: 1.15;
  height: 26px;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

### Kaarten met variabele content uitlijnen
Flex column met vaste hoogtes per zone zodat CTA/prijs/image altijd op dezelfde y-positie staan:

```css
.id20-card { display: flex; flex-direction: column; }
.id20-card-img   { height: 150px; }
.id20-card-title { height: 36px; -webkit-line-clamp: 2; overflow: hidden; }
.id20-card-price { height: 22px; }
.id20-card-cta   { margin-top: auto; }
```

### Horizontaal scrollbare vergelijkingstabellen
- Eén scroll-container voor de hele tabel — niet per rij een aparte scroll
- `position: sticky; left: 0` op de label-kolom
- `position: sticky; top: 0` op de productrij
- Geen `scroll-snap` als kolommen niet 100% breed zijn
- Identieke kolombreedte op productrij én spec-rijen
- Geen JS scroll-sync tussen meerdere overflow-containers

### Sticky werkt niet als een parent `overflow: hidden` heeft
Check de hele parent-keten. Sticky binnen een geneste `overflow: auto` werkt vaak niet zoals verwacht.

## Cross-row alignment in tabellen
`.row { display: flex; align-items: stretch; }` + `.cell { display: flex; align-items: flex-start; }` zorgt voor gelijke baseline, ook als één cel meer content heeft.

## Default states & user-initiated UI
Floating panels, modals en paneeltjes starten **standaard ingeklapt**. Gebruiker opent het bewust — voorkomt dat content op de pagina afgedekt wordt na pageload.

## Kleur, font-weight en spacing
- Pak kleuren letterlijk uit het design (Figma/screenshot) — niet gokken
- Font-weight verschilt vaak tussen 500 en 700; klein visueel verschil, maar merkbaar. Altijd checken.
- Margin/padding één-op-één uit het design — niet "ongeveer hetzelfde"

## Scrollbar als affordance
Wanneer horizontaal scrollen mogelijk is maar visueel niet duidelijk, toon een dunne zichtbare scrollbar als hint. Op iOS via `::-webkit-scrollbar`.

## Working version workflow

- **current-version** = snapshot van wat nu live staat. Wordt **NOOIT** direct aangepast.
- **working-version** = alle aanpassingen en iteraties komen hier.
- Pas wanneer een working version goedgekeurd en live is, wordt current-version overschreven.

## Werkwijze bij iteratieve feedback
- Maak per feedback-ronde één gerichte fix, geen gelegenheidsrefactor
- Test direct in de browser na elke wijziging
- Bij visuele mismatches: vraag om een screenshot, niet om een beschrijving

## Testen

- Test via Tampermonkey of browser-snippet
- Controleer op console errors
- Test op desktop én mobiel (minimaal 375px en 768px+)
- Verifieer dat het INIT event zichtbaar is in de dataLayer (bij Convert/Optimizely/Kameleoon)
- Controleer afbeeldingen op beide viewports
- **Test echte interacties** (klikken, swipen, scrollen) — niet alleen visueel inspecteren
