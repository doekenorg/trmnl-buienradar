# Buienradar — TRMNL Plugin

A [TRMNL](https://trmnl.com) plugin that shows live Dutch weather from
[Buienradar](https://www.buienradar.nl) on an e-ink TRMNL device: current
conditions, wind, high/low temperatures, and a 4-day forecast, drawn with custom
monochrome weather icons. Built against TRMNL framework **v3.1.1** (plugin id `353457`).

## Features

- Current conditions with a custom monochrome weather icon, wind, and today's hi/lo.
- 4-day forecast, one card per day.
- Custom e-ink icon set (single inlined SVG sprite, `currentColor`-driven so it
  inverts cleanly in dark mode).
- Four device layouts: **full**, **half horizontal**, **half vertical**, and **quadrant**.
- Per-device and responsive handling (OG TRMNL, TRMNL X, portrait mounts, 1/2/4-bit).
- °C / °F toggle.
- Optional radar panel (paste your own crop HTML into the config).
- Pick any of 39 Buienradar measurement stations.

## Data

Polled from the public Buienradar feed:

```
GET https://data.buienradar.nl/2.0/feed/json
```

Refreshes every **15 minutes**.

## Configuration

User-facing fields (defined in `src/settings.yml`):

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| Weather station | select (39 stations) | Groningen (Eelde), `6280` | Buienradar measurement station nearest you. |
| Show station name in footer | boolean | on | Include the station name in the bottom title bar. |
| Show update time in footer | boolean | on | Include the latest measurement time in the bottom title bar. |
| Temperature unit | select (`c` / `f`) | Celsius | Show temperatures in Celsius or Fahrenheit. |
| Radar panel HTML | code (optional) | — | Custom HTML shown beside current conditions in the quadrant view. Blank hides it. |

## Repository layout

```
trmnl-buienradar/
├── AGENTS.md          # Authoritative project + contributor rules — read before editing
├── CLAUDE.md          # Imports AGENTS.md
├── src/               # The deployed plugin (synced with TRMNL)
│   ├── settings.yml   # Polling config + custom fields (the config form)
│   ├── shared.liquid  # Prepended to every view: config assigns, icon sprite, shared partials
│   ├── full.liquid
│   ├── half_horizontal.liquid
│   ├── half_vertical.liquid
│   └── quadrant.liquid
└── assets/            # Source art + reference (NOT deployed)
    ├── icons-original.svg    # Colorful, un-merged icon master — never edit; rebuild from this
    ├── buienradar-icons.svg  # Processed sprite that gets inlined into shared.liquid
    └── buienradar.svg        # Logo
```

`shared.liquid` is prepended to every view before render. It holds the config
assigns, the icon sprite (declared once to stay under TRMNL's ~100 KB per-template
limit), and the reusable partials (`weather_icon`, `temp`, `now_block`, `now_bar`,
`forecast_card`, `text_block`, `titlebar`). The view files stay thin — layout
markup plus `{% render %}` calls.

## Development

The `src/` folder follows the layout expected by TRMNL's local dev tool
([trmnlp](https://github.com/usetrmnl/trmnlp)) and the import/export ZIP, so the
plugin can be previewed locally and synced to TRMNL.

The icon sprite is regenerated from `assets/icons-original.svg` through an svgo
pipeline (recolor accents to `currentColor`, two-tone clouds, drop night symbols,
then re-inline into `shared.liquid`). The exact svgo flags and reasoning live in
`AGENTS.md` — follow them; the defaults strip the icons.

**Before editing, read `AGENTS.md`.** It is the authoritative best-practices doc:
utility-class-only styling (no inline `style=""`), responsive/device prefixes,
the Liquid config conventions, the `settings.yml` schema, and the icon build rules.

## Author

By [Doeke Norg](https://doeken.org).
