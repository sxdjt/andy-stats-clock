# Multi-Day History Rings

Any history or consumption layer can render multiple concentric rings - one per
day - by setting `days` to a value greater than 1. The outermost ring is always
today; each inner ring is one day older.

This makes it easy to compare today's energy price or consumption pattern
against previous days at a glance, using the same entity and color scale.

---

## How it works

- Each day fetches its own hourly buckets from the HA history API, cached
  independently (5-minute TTL for today, 10-minute TTL for past days).
- All rings in the group share the same min/max scale so colors are directly
  comparable across days.
- Older rings fade progressively so today's data is visually dominant.
- Stats markers (min/max/avg) are shown only on today's ring.
- "History day" and "Keep values across midnight" are not available in
  multi-day mode; set `days: 1` to use those features instead.

---

## Configuration reference

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `days` | integer 1-7 | `1` | Number of day rings. 1 = single-day (existing behaviour). |
| `day_fade` | boolean | `true` | Reduce opacity on older day rings. |
| `day_fade_step` | float 0-1 | `0.25` | Opacity multiplier reduction per day. Ring N opacity = `base * max(0, 1 - N * step)`. |
| `day_labels` | boolean | `false` | Show a text label on each ring. |
| `day_label_format` | `relative` / `date` | `relative` | `relative` = "Today", "Yesterday", "2d ago". `date` = "Jan 15". |
| `day_label_position` | `right` / `left` / `top` / `bottom` | `right` | Clock position of the label on each ring. |
| `day_label_font_size` | number | `3` | SVG font size for day labels. |
| `day_label_color` | color string | `rgba(255,255,255,0.7)` | Day label text color. |

---

## Examples

### 3-day energy price comparison

Show today, yesterday, and two days ago as concentric rings. Older rings fade
to 50% and 25% of the base opacity respectively.

```yaml
type: custom:andy-stats-clock
clock_mode: 24h
layers:
  - id: price_multiday
    type: price
    entity: sensor.nordpool_kwh_se3_sek_3_10_025
    price_source: array
    attribute: today
    thickness: 6
    color_mode: intervals
    intervals:
      - color: "#28c76f"
        max: 0.5
      - color: "#ff9f43"
        max: 1.0
      - color: "#ea5455"
        max: 999
    days: 3
    day_fade: true
    day_fade_step: 0.25
```

### 7-day consumption with date labels

```yaml
type: custom:andy-stats-clock
clock_mode: 24h
layers:
  - id: consumption_week
    type: consumption
    entity: sensor.electricity_consumption_kwh
    thickness: 5
    color_mode: gradient
    gradient:
      - "#1a1a2e"
      - "#16213e"
      - "#0f3460"
      - "#533483"
    days: 7
    day_fade: true
    day_fade_step: 0.12
    day_labels: true
    day_label_format: date
    day_label_position: right
    day_label_font_size: 3
    day_label_color: "rgba(255,255,255,0.6)"
```

### Single day (unchanged behaviour)

Omitting `days` or setting it to `1` preserves all existing single-day
behaviour, including "History day" offset and "Keep values across midnight".

```yaml
layers:
  - id: price_today
    type: price
    entity: sensor.nordpool_kwh_se3_sek_3_10_025
    price_source: array
    attribute: today
    days: 1                    # default - single ring, all existing options available
    keep_across_midnight: true
    fade_previous_day: true
    fade_previous_day_opacity: 0.3
```

---

## Visual editor

Multi-day settings appear in the layer editor under **Multi-day comparison**,
visible for history and consumption layer types. When `days` is set above 1,
the "History day" and "Keep values across midnight" sections are hidden
automatically since they do not apply in multi-day mode.
