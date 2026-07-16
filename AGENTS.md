# Buienradar — TRMNL Plugin

## Project

This is the TRMNL plugin "Buienradar" (`settings.yml` id `353457`). It shows current conditions, wind, hi/lo temperatures, and a 4-day forecast for any Dutch Buienradar weather station on an e-ink TRMNL device. It ships with custom monochrome weather icons. Data comes from polling `https://data.buienradar.nl/2.0/feed/json` on a 15-minute refresh. It is built against TRMNL framework v3.1.

## Layout / Structure

The repo root contains this `AGENTS.md`, a `CLAUDE.md` (which imports it), a `src/` folder, and an `assets/` folder.

### `src/` — the live plugin

These files are pasted into or synced with TRMNL.

- `src/settings.yml` — plugin config: polling, refresh, `framework_version`, and `custom_fields` (the user-facing config form).
- `src/shared.liquid` — prepended to EVERY view before render. It holds the config assigns (read from `trmnl.plugin_settings.custom_fields_values`), the hidden icon sprite (declared once), and all reusable partials defined via `{% template name %}…{% endtemplate %}`.
- `src/full.liquid`, `src/half_horizontal.liquid`, `src/half_vertical.liquid`, `src/quadrant.liquid` — the four device layouts. Each is just layout markup plus `{% render %}` calls. Keep them thin.

### Partials

Partials live in `src/shared.liquid`:

- `weather_icon(code, size)` — renders one sprite icon.
- `temp(c, unit, with_unit)` — formats a temperature.
- `now_block` — the vertical current-conditions block (full and both halves).
- `now_bar` — the horizontal current-conditions bar (quadrant).
- `forecast_card(day, dagen, icon_size, …)` — one forecast day card.
- `text_block(heading, body)` — a labelled text block.
- `titlebar(station, show_name, show_time)` — the header.

`{% render %}` is scope-isolated: a partial ONLY sees the params you pass it, not the shared `assign`s. Pass every variable a partial needs. Nested renders are used (e.g. `now_block` renders `weather_icon` and `temp`).

### `assets/` — source art and reference

None of this is deployed.

- `assets/icons-original.svg` — the colorful, un-merged icon MASTER (see Icons). Never edit it.
- `assets/buienradar-icons.svg` — the processed sprite that gets inlined into `shared.liquid`.
- `assets/buienradar.svg` — the logo.
- `assets/form-fields.yml` — form-fields reference.

## Best Practices (HARD RULES)

1. **Utility classes only. NO inline `style=""`.** TRMNL discourages inline styles. Use the native equivalents: width/height `w--{n}` / `h--{n}`, `w--full` / `h--full`, arbitrary `w--[Npx]`, and percentage-of-`.layout` via container-query units `w--[Ncqw]` / `h--[Ncqh]` (N from 0–100). Hide/show with `hidden` (`display:none`) / `visible` (`display:block`). There is no grid-rows utility — for a 2D equal grid, use grid columns for width plus a flex column of `grow` rows for height.

2. **Device / responsive states.** Prefixes: size `sm:` (≥600), `md:` (≥800), `lg:` (≥1024); orientation `portrait:` (landscape is the unprefixed default — portrait is the odd one out); bit-depth `1bit:` / `2bit:` / `4bit:`; and `dark:`. Mapping: **base = the original/OG TRMNL (smaller screen) in landscape**, **`lg:` = TRMNL X (the larger 4-bit screen)**, and `portrait:` = a device mounted portrait. Combine in the order `dark:size:orientation:bit-depth:utility` (e.g. `md:portrait:4bit:gap--large`). **Never use `landscape:`-prefixed classes**, even though the shipped framework CSS contains them and the official docs mention them — this rule is correct and the docs are not. The runtime never sets a `screen--landscape` class, so `landscape:` padding/margin utilities are dead, while the `landscape:` flex rules match via `:not(.screen--portrait)` and therefore leak into `lg:` landscape with tie-breaking specificity that `lg:landscape:` overrides cannot reliably win. To target OG-landscape only: style the base, then undo with `portrait:` and `lg:` (e.g. `flex--row flex--between portrait:flex--col lg:flex--col`). When verifying that a prefixed class exists in the framework CSS, beware that most rules live in comma-grouped selector lists — grep for the bare class name (e.g. `portrait\\:px--2`), not for a `{` right after it.

3. **Specificity gotcha.** More modifiers means higher specificity. A prefixed display utility like `lg:flex` carries a default ~10px gap that OUTRANKS an unprefixed `gap--xlarge`. When you set display with a prefix, set the gap at the same prefix level (`lg:gap--xlarge`).

4. **Equal sizing.** Equal WIDTH → grid columns (`grid--cols-N`, 1fr each, content-independent). Equal HEIGHT → flex `grow` rows plus cards with `h--full` (the `fill` param on `forecast_card`). `grow` alone gives content-width, NOT equal width.

5. **Icons.** All sprite fills use `fill="currentColor"` so icons follow the ambient text color (black normally, white when inverted); clouds use `fill-opacity:.55` for a two-tone that also inverts. The icon `<svg class="image">` carries NO color so it inherits. Code lookup: parse `fullIconUrl | split:"/" | last | split:"." | first | upcase` to match a `<symbol id>`. Night codes are the day letter doubled; the Now block adds `| slice: 0` so a night reading falls back to the day icon (the night symbols were dropped). The sprite root MUST be hidden (`class="hidden"` / `display:none`) — `<use>` still resolves from it.

6. **Icon build pipeline** (when re-generating `buienradar-icons.svg`): start from `icons-original.svg` (the colorful, un-merged MASTER — never edit it). Recolor accents to `currentColor` (NOT `#000` — svgo strips black as the default and the icon then stops following `currentColor`), and recolor clouds to `currentColor` plus `fill-opacity:.55`. Drop the double-letter night symbols. Run svgo with `mergePaths:false` (force-merge collapses the sun's duplicate ray paths under evenodd and the rays vanish), `removeHiddenElems:false` plus `removeUselessDefs:false` (otherwise symbols with no in-file `<use>` get stripped), `cleanupIds:false` (keep the symbol ids), `convertColors:false`, and precision 2. Then re-inline into `shared.liquid` with the root `<svg>` hidden.

7. **Config in Liquid.** Read `trmnl.plugin_settings.custom_fields_values`. Boolean fields arrive as the STRINGS `"true"` / `"false"` — normalize them before `{% if %}`. Numeric select values: coerce with `| plus: 0` before a `where` match. Provide defaults with `| default:`.

8. **`settings.yml` / form-builder schema.** Use only the documented top-level keys (`name`, `strategy`, `polling_*`, `refresh_interval`, `framework_version`, `dark_mode`, `no_screen_padding`, `custom_fields`, `id`). Field types: `select`, `string`, `number`, `code` (a textarea — use `rows:`), `date`, `boolean`, `author_bio`. There is NO `text` type. `author_bio` is itself a FIELD (it renders an About card), not a top-level key. select `options` are `- Label: value` at the same indent as `options:`.

9. **Per-template ~100 KB limit.** The icon sprite is large, so it lives ONCE in `shared.liquid` (which is prepended to all views) rather than being duplicated per view. Keep the view files thin.

10. **When given a documentation link, read it (verbatim if needed) — do not guess a schema or API.**

## Workflow Note

TRMNL's local dev tool (trmnlp) and the import/export ZIP expect the plugin files in a `src/` folder, which this repo uses.
