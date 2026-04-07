# Design: The Lonely Earth

## Goal

Show a general audience that among the 1,174 known exoplanets in this dataset, planets resembling Earth in size and temperature are extraordinarily rare — only 8 out of 1,174.

## Audience

General public reader. Curious about space, not technical. Assumes no knowledge of astronomy terminology. Earth is the universal reference point — every measurement is relative to it.

## Story Hypothesis

Most exoplanets look nothing like Earth. The universe we've observed so far is dominated by gas giants and scorching hot worlds. Planets in Earth's size and temperature range are a tiny minority. The reader should walk away thinking: "We really are unusual."

## Visualization Choice

**Scaled bubble chart** — each planet is a circle whose area is proportional to its radius (in Earth radii). Planets are grouped into size categories and colored by equilibrium temperature.

### Why bubbles?

- Size *is* the story — encoding it as visual area makes the diversity visceral and immediate
- A general audience doesn't need to read axes; big circle = big planet, small circle = small planet
- The contrast between the dominant Jupiter-sized cluster and the tiny Earth-sized cluster delivers the "lonely" feeling without explanation

### Alternatives considered

| Option | Why rejected |
|---|---|
| Strip plot / beeswarm | Shows distribution shape well, but size is encoded as position — more abstract, less emotional impact |
| Histogram with Earth marker | Clean and readable, but too clinical; loses individual planets and the visceral size comparison |

## Layout

Planets are arranged in **8 columns by size category**, left to right from smallest to largest:

```
Sub-Earth | Earth-size | Super-Earth | Mini-Neptune | Neptune | Sub-Saturn | Jupiter | Super-Jupiter
  (<0.8)   (0.8-1.25)   (1.25-2.0)     (2-4)        (4-6)     (6-10)     (10-15)     (>15)
   13          47           111           261           48        113         414         167
```

Within each column, circles are packed vertically. The reader scans left to right and sees:
- A sparse left side (few small planets)
- A massive right side (hundreds of giants)
- Earth's column is nearly empty compared to the giant clusters

### Text diagram

```
  The Lonely Earth
  ================

  Sub-    Earth  Super-  Mini-   Neptune  Sub-    Jupiter  Super-
  Earth   size   Earth   Neptune -like    Saturn  -like    Jupiter
  ─────── ────── ─────── ─────── ──────── ─────── ──────── ────────

   .        .      o       O       O        O       OO       OOO
   .        .      o       O                 O       OO       OOO
   .       [.]     o       OO                O       OO       OO
            .      oo      OO                O       OO       OO
            .      oo      OO                        OO       OO
                   oo      OO                        OO       OO
                   oo      O                         OO       O
                   o       O                         OO
                           O                         OO
                           O                         O
                                                     O
                                                     O
                                                     O

  [.] = Earth (highlighted)       Circle size = planet radius
  Warm colors = hot planets       Cool colors = mild temperatures

  "Only 8 of 1,174 exoplanets are similar to Earth
   in both size and temperature."

  ┌─────────────────────────────────────────────────┐
  │  Caveat: Smaller planets are harder to detect.  │
  │  The rarity is partly real, partly a limitation │
  │  of current instruments.                        │
  └─────────────────────────────────────────────────┘
```

## Color Encoding

Equilibrium temperature mapped to a sequential gradient:

- **Blue** (~0-300K): Cold worlds
- **Teal/Green** (~300-600K): Mild (Earth-like range ~228-347K falls here)
- **Orange** (~600-1500K): Hot
- **Red** (~1500K+): Scorching

The 8 Earth-like candidates naturally cluster in the teal/green range, visually distinguishing them from the sea of orange/red giants.

## Highlight Strategy

1. **Earth** — labeled, outlined with a bright ring or glow, distinct from all other points
2. **8 Earth-like candidates** — secondary highlight (labeled on hover or with small callout), same temperature color but with a visible border
3. **All other planets** — slightly reduced opacity so the highlights pop

The 8 Earth-like planets (radius 0.8-1.5, temp 200-350K):
- Gliese 12 b (0.96R, 315K)
- K2-3 d (1.46R, 305K)
- Kepler-62 f (1.41R, 208K)
- LP 890-9 c (1.37R, 272K)
- TOI-2095 b (1.25R, 347K)
- TOI-2095 c (1.33R, 297K)
- TRAPPIST-1 c (1.10R, 311K)
- TRAPPIST-1 e (0.92R, 228K)

## Key Data Columns

| Column | Role |
|---|---|
| `pl_rade` | Bubble size (planet radius in Earth radii) |
| `pl_eqt` | Bubble color (equilibrium temperature in Kelvin) |
| `pl_name` | Labels for highlighted planets |
| `pl_bmasse` | Tooltip detail (planet mass in Earth masses) |

## Columns Excluded

| Column | Why excluded |
|---|---|
| `discoverymethod` | 96% are Transit — not enough variation to be interesting here |
| `ra`, `dec` | Sky coordinates — irrelevant to the size story |
| `sy_dist` | Distance from Earth — interesting but dilutes the core message |
| `pl_orbper`, `pl_orbsmax` | Orbital properties — separate story |
| `st_teff`, `st_rad`, `st_mass`, `st_met` | Stellar properties — too technical for this audience |
| `disc_year` | Discovery timeline — separate story |
| `sy_pnum` | Multi-planet systems — separate story |
| `star_id`, `hostname` | Identifiers only |
| `pl_eqt_computed` | Internal flag |

## Risks

1. **Visual clutter** — 1,174 circles with 100x size range. Mitigation: group by category, reduce opacity on non-highlighted planets, cap minimum visual size so tiny planets remain visible.
2. **Detection bias misinterpretation** — Reader may conclude Earth-like planets don't exist, rather than that they're hard to find. Mitigation: caveat panel explaining observational limits.
3. **Bubble area perception** — Humans are bad at comparing circle areas. Mitigation: size categories provide structure; exact values available in tooltips. The story relies on cluster size (count), not individual bubble comparison.
4. **Temperature gradient accessibility** — Blue-to-red gradients can be problematic for colorblind readers. Mitigation: use a perceptually uniform palette (e.g., viridis-inspired) and don't rely on color alone — temperature also shown in tooltips.
