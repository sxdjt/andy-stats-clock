# Changelog

## [Unreleased] - sxdjt fork

### Changed

- Replaced deprecated `ha-select` / `mwc-list-item` dropdown pattern throughout
  the visual editor with `ha-selector` (select type), matching current Home
  Assistant standards. Affected fields: clock mode, layer type, value source,
  history day, color mode, center layer type, bottom layer type, day label
  format, day label position.
- Replaced deprecated boolean `ha-select` fields with `ha-formfield` + `ha-switch`
  throughout the visual editor. Affected fields: show hour labels, show minute
  ticks, hands share center pivot, enable hour/minute/second hand, center show
  icon, bottom show icon.

### Added

- **Multi-day history rings** (reimplementation of PR #4 - original local repo
  was accidentally deleted): a history/consumption layer can now be configured
  with `days: N` (1-7) to render N concentric rings, one per day. The outermost
  ring is today; inner rings are progressively older days.
  - New `_getHistoryForDay(lc, hass, stateObj, dayOffset)` method fetches and
    caches one day's hourly bucket data per ring. Cache keys are scoped per
    entity, layer, day offset, and segment count. TTL is 5 minutes for today,
    10 minutes for past days.
  - `_buildLayerData()` pre-expands multi-day layers into an expanded list before
    radius assignment, so each day-ring occupies its own concentric slot. A
    second pass shares min/max across all rings in a multi-day group so the
    color scale is consistent across days.
  - `day_fade` (default `true`) and `day_fade_step` (default `0.25`) - fade
    opacity applied per-ring for older days so recent data is visually dominant.
  - Stats markers (min/max/avg) are suppressed on non-today rings to avoid
    visual clutter.
  - `day_labels` - optional SVG text label on each ring showing relative ("Today",
    "Yesterday", "2d ago") or absolute date ("Jan 15") format, configurable via
    `day_label_format`, `day_label_position`, `day_label_font_size`,
    `day_label_color`.
  - New `_renderDayLabel(cfg, layer)` method produces the SVG label element.
  - "History day" selector and "Keep values across midnight" are hidden in the
    visual editor when `days > 1`, as they are not applicable in multi-day mode.
  - Multi-day editor section added under history/consumption layers with controls
    for: days count, fade toggle, fade step, day labels toggle, label format,
    label position.
