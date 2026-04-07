# Plan: The Lonely Earth — Exoplanet Visualization

## Summary

Build a single-page interactive bubble chart showing 1,174 exoplanets as circles scaled to their radius, grouped into 8 size categories, colored by equilibrium temperature. Earth and 8 Earth-like candidates are highlighted to tell the story: planets like ours are extraordinarily rare.

**Audience:** General public, non-technical.
**Format:** Single HTML file (HTML + CSS + JS inline), zero external dependencies. Dark space theme.
**Data:** `exoplanets.csv` — 1,174 rows, loaded via `fetch()` at runtime.

See `DESIGN.md` for full design rationale, alternatives considered, and risk mitigations.

---

## Phase 1: Data Layer

**Goal:** Load, parse, classify, and expose the dataset for rendering.

### Deliverables

1. `loadCSV()` — fetch `exoplanets.csv`, parse with a CSV parser that handles quoted fields
2. `classifyPlanets()` — assign each planet to one of 8 size categories based on `pl_rade`:
   - Sub-Earth: < 0.8
   - Earth-size: 0.8–1.25
   - Super-Earth: 1.25–2.0
   - Mini-Neptune: 2.0–4.0
   - Neptune-like: 4.0–6.0
   - Sub-Saturn: 6.0–10.0
   - Jupiter-like: 10.0–15.0
   - Super-Jupiter: > 15.0
3. `identifyEarthLike()` — flag the 8 planets with radius 0.8–1.5 AND temperature 200–350K:
   - Gliese 12 b, K2-3 d, Kepler-62 f, LP 890-9 c, TOI-2095 b, TOI-2095 c, TRAPPIST-1 c, TRAPPIST-1 e
4. Data structure: array of planet objects with properties: `name`, `radius`, `mass`, `temperature`, `category`, `isEarthLike`

### Verification

- Console log confirms 1,174 planets loaded
- Console log confirms category counts match: Sub-Earth 13, Earth-size 47, Super-Earth 111, Mini-Neptune 261, Neptune-like 48, Sub-Saturn 113, Jupiter-like 414, Super-Jupiter 167
- Console log confirms exactly 8 Earth-like planets identified
- No parsing errors on any row

---

## Phase 2: Page Structure and Theme

**Goal:** Build the HTML skeleton and dark space theme.

### Deliverables

1. HTML structure:
   - Header: title "The Lonely Earth", subtitle about exoplanet diversity
   - Main: SVG container for the bubble chart
   - Footer: caveat panel about detection bias, legend
2. CSS custom properties in `:root`:
   - `--bg-deep`: deep navy/black (#0a0e1a or similar)
   - `--bg-card`: slightly lighter panel background
   - `--text-primary`: white/light gray
   - `--text-muted`: dimmed text for secondary info
   - `--accent`: highlight color for Earth marker (bright gold/white)
   - Temperature colors: `--temp-cold`, `--temp-mild`, `--temp-hot`, `--temp-scorching`
3. Typography: clean sans-serif (system font stack), large title, readable body text
4. Layout: centered container, max-width ~1200px, responsive padding

### Verification

- Page loads with dark background, title visible, SVG container has defined dimensions
- No horizontal scroll on viewport widths 800px–1400px
- Footer caveat text is readable

---

## Phase 3: Bubble Chart — Core Rendering

**Goal:** Render all 1,174 planets as SVG circles, grouped by size category, scaled by radius.

### Deliverables

1. SVG canvas sized to fit all 8 category columns
2. Category columns laid out left-to-right with labels underneath:
   - Each column gets proportional width (wider columns for categories with more planets)
   - Column header labels: category name and count
3. Circle sizing:
   - Area proportional to `pl_rade`
   - Minimum visual radius of 3px so the smallest planets (0.31 Earth radii) remain visible
   - Maximum visual radius capped so the largest (33.6 Earth radii) don't overwhelm the layout
   - Use a sqrt scale (since area = pi*r^2, visual radius = sqrt(pl_rade) * scaleFactor)
4. Circle packing within each column:
   - Simple vertical stacking or force-based packing within the column bounds
   - No overlapping circles
5. Base styling: all circles at ~40% opacity initially

### Verification

- All 1,174 circles are visible in the SVG
- Circles don't overflow their category columns
- Clear visual distinction between category columns (sparse left, dense right)
- Smallest circles are at least 3px radius and visible
- Largest circles don't dominate more than ~15% of the SVG width

---

## Phase 4: Color Encoding

**Goal:** Color each circle by equilibrium temperature.

### Deliverables

1. Temperature-to-color mapping function:
   - 0–300K → Blue (#4a90d9 range)
   - 300–600K → Teal/Green (#2dd4a8 range)
   - 600–1500K → Orange (#f59e0b range)
   - 1500K+ → Red (#ef4444 range)
   - Use smooth interpolation, not hard steps
2. Handle missing temperature values: use a neutral gray
3. Apply fill colors to all circles

### Verification

- Circles show a visible range of colors across the chart
- Earth-like candidates (200–350K) are clearly in the blue-to-teal range
- Jupiter-like column is predominantly orange/red (most hot Jupiters)
- No circles are unstyled/black (missing values get gray)

---

## Phase 5: Highlights and Annotations

**Goal:** Make Earth and the 8 Earth-like candidates visually prominent. Add the headline stat.

### Deliverables

1. **Earth highlight:**
   - Earth is not in the dataset (it's not an exoplanet). Add a reference marker in the Earth-size column at radius=1.0, temp=255K
   - Bright ring/glow effect (CSS filter or SVG filter for glow)
   - Permanent label: "Earth" with a connecting line if needed
2. **Earth-like candidates:**
   - White or bright border on the 8 matching circles
   - On hover: show planet name, radius, temperature
   - Subtle pulsing animation or distinct border style
3. **All other planets:** remain at reduced opacity (~40%) so highlights pop
4. **Headline stat:** prominent text above or overlaying the chart:
   - "Only 8 of 1,174 exoplanets resemble Earth in size and temperature"
   - Large font, high contrast
5. **Category count labels** below each column showing the count

### Verification

- Earth marker is immediately visible without scrolling or hovering
- The 8 Earth-like planets are visually distinguishable from surrounding circles
- Headline stat is readable and prominent
- Hovering over an Earth-like candidate shows its name and stats
- The overall visual impression: lots of big colorful dots, a few tiny highlighted ones

---

## Phase 6: Interactivity and Tooltips

**Goal:** Add hover tooltips for all planets, not just the highlighted ones.

### Deliverables

1. Tooltip on hover for any circle:
   - Planet name
   - Radius (in Earth radii, 2 decimal places)
   - Mass (in Earth masses, 1 decimal place)
   - Temperature (in Kelvin, with human-friendly label: "Cold", "Mild", "Hot", "Scorching")
   - Size category
2. Tooltip styling: dark card with light text, positioned near cursor, doesn't overflow viewport
3. Hover state: circle opacity increases to 100%, slight scale-up

### Verification

- Hovering over any circle shows a tooltip with correct data
- Tooltip doesn't get cut off at viewport edges
- Tooltip disappears when mouse leaves the circle
- Tooltip data matches the CSV source for spot-checked planets

---

## Phase 7: Legend and Caveat Panel

**Goal:** Provide the reader with context to interpret the chart.

### Deliverables

1. **Color legend:** horizontal gradient bar showing temperature scale from Cold (blue) to Scorching (red), with Kelvin labels at key points
2. **Size legend:** 2-3 reference circles showing example sizes ("Earth = 1.0", "Neptune = 3.9", "Jupiter = 11.2") so the reader can calibrate
3. **Caveat panel** at the bottom:
   - Text: "Smaller planets are harder to detect with current instruments. The rarity of Earth-like worlds in this data is partly real and partly a limitation of how we find exoplanets."
   - Styled as a subtle card/callout, not a warning banner
4. **Data source line:** "Data: NASA Exoplanet Archive" (or appropriate attribution)

### Verification

- Color legend gradient is smooth and labels are readable
- Size reference circles are correctly scaled relative to the chart
- Caveat text is visible but doesn't compete with the main visualization
- Attribution is present

---

## Constraints

- **Single HTML file** — all HTML, CSS, and JavaScript inline. No external libraries, no build step.
- **Data file** — `exoplanets.csv` in the same directory, loaded via `fetch()`
- **No Google Fonts or CDN dependencies** — use system font stack
- **Minimum viable viewport:** 800px wide. Doesn't need to be mobile-responsive, but shouldn't break.
- **Browser target:** modern browsers (Chrome, Firefox, Safari). ES6+ is fine.

## File Structure

```
Exercise 2/
  index.html          ← single-file implementation
  exoplanets.csv      ← data (already exists)
  DESIGN.md           ← design document (already exists)
  PLAN.md             ← this file
```

## Implementation Order

Build and verify each phase before moving to the next. If Phase 3 (core rendering) doesn't look right, don't proceed to Phase 4 — fix it first. Phases 6 and 7 are polish and can be simplified if time is short.
