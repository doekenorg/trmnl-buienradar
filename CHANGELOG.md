# Changelog

All notable changes to this plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.1] - 2026-09-29

### Fixed

- Buienradar changed their API: the 5-day forecast no longer starts with today, but with tomorrow.
  Because of this, the high/low temperature shown next to the current conditions was actually
  tomorrow's, and tomorrow was missing from the forecast cards. Today and the forecast days are now
  matched by date, so the forecast always starts at tomorrow, and the high/low is hidden when
  Buienradar has no data for today.

## [1.1.0] - 2026-07-16

### Added

- Show the short-term forecast text on both half layouts in portrait.

### Changed

- Plugin renamed to "Buienradar Station".
- Better portrait rendering across all layouts.
- Bigger current-day high/low and label, with the low temperature shown in a lighter shade.
- Forecast cards use equal temperature sizes, with the night temperature in a lighter shade.
- Half horizontal: the weather icon now sits above the current temperature, and the text is smaller
  on the original TRMNL.
- Half vertical: forecast cards are laid out as rows on the original TRMNL in landscape.
- Quadrant: the radar panel only shows on the TRMNL X in landscape.

### Fixed

- Quadrant: the radar panel no longer shows on the TRMNL X in portrait.

## [1.0.0] - 2026-06-27

### Added

- First release: current conditions, wind, high/low temperatures and a 4-day forecast for any
  Buienradar weather station, with custom monochrome weather icons.
- Full, half horizontal, half vertical and quadrant layouts.
- Settings for the station, °C or °F, showing the station name and time, and an optional radar
  panel.

[Unreleased]: https://github.com/doekenorg/trmnl-buienradar/compare/v1.1.1...HEAD
[1.1.1]: https://github.com/doekenorg/trmnl-buienradar/compare/v1.1.0...v1.1.1
[1.1.0]: https://github.com/doekenorg/trmnl-buienradar/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/doekenorg/trmnl-buienradar/releases/tag/v1.0.0
