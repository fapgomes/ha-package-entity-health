# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-06-29

### Changed

- **Frozen detection is now opt-in.** Detecting a "frozen" value from a
  timestamp cannot distinguish a real fault from a sensor that is naturally
  static (battery %, an idle plug's energy total, cloud precipitation at 0 mm,
  entity counters, a vacuum's lifetime totals…), so scanning every sensor only
  produced false positives. `sensor.frozen_entities` now checks **only** the
  entities you explicitly list, instead of iterating over all `states.sensor`.
  A genuinely dead device is still caught by the *Unavailable* monitor.
- Kept `last_changed` as the stuck-value metric. `last_reported` does not help:
  the MQTT integration discards identical payloads, so it never advances on a
  repeated value and collapses onto `last_changed`
  ([home-assistant/core#121978](https://github.com/home-assistant/core/issues/121978)).

### Added

- `group.watched_frozen_entities` and the `watched_frozen` label — the opt-in
  watchlist that is now the only source for frozen detection.

### Removed

- `group.ignored_frozen_entities` and the `ignored_from_frozen` label — no longer
  needed under the opt-in model (just don't add an entity to the watchlist).
- The automatic scan of all sensors, the `state_class` filter, and the
  intermediate MQTT exclusion (`integration_entities('mqtt')`).

## [1.1.1] - 2026-06-24

### Changed

- Translated the remaining Portuguese strings to English for consistency: the
  frozen automation aliases (*Aviso: entidades com valor parado* → **Frozen
  Entities Alert**, *Limpar aviso de entidades com valor parado* → **Clear
  Frozen Entities Alert**), the input helper names, and the in-file comments
  (header notes and CONFIG block). The user-facing notification text remains in
  Portuguese.
- Raised the default frozen thresholds: `frozen_default_limit_hours` 6 → **12**,
  `frozen_tight_limit_hours` 3 → **10** (helper `initial` values and the template
  fallbacks).

## [1.1.0] - 2026-06-24

### Added

- `group.frozen_entities` — a real group of the frozen sensors' raw entity ids,
  so they can be used in cards and `auto-entities` just like
  `group.unavailable_entities`.
- *Update Frozen Entities Group* automation — rebuilds the group every minute
  (and on `group.reload`).
- `entity_ids` attribute on `sensor.frozen_entities` — the raw ids (age suffix
  stripped) that feed the group. Derived from the existing `entities` list, so
  the detection still runs only once.
- Configuration via input helpers at the top of the package, replacing the
  hard-coded values:
  - `input_number.frozen_default_limit_hours` (default 6),
    `input_number.frozen_tight_limit_hours` (default 3),
    `input_number.frozen_notify_delay_minutes` (default 15),
    `input_text.frozen_notify_service` (default `notify.calvin`).

### Changed

- The frozen notification's external service is now read from
  `input_text.frozen_notify_service` instead of a hard-coded `notify.calvin`,
  and is skipped entirely when that field is empty.

## [1.0.0] - 2026-06-24

### Added

- Initial release of the **Entity Health** Home Assistant package.
- **Unavailable Entities** monitor (v2.4, based on
  [jazzyisj/unavailable-entities-sensor](https://github.com/jazzyisj/unavailable-entities-sensor)):
  - `sensor.disabled_device_entities` — count and list of disabled device entities.
  - `sensor.unavailable_entities` — count of `unknown` / `unavailable` entities.
  - `group.unavailable_entities` rebuilt every minute via the
    *Update Unavailable Entities Group* automation, with extensive `rejectattr`
    filtering of noisy patterns.
  - *Unavailable Entities Notification* automation (persistent notification).
  - `group.ignored_entities` for manual exclusions.
- **Frozen (stuck-value) Entities** monitor (v1.0):
  - `sensor.frozen_entities` — detects sensors whose value (`last_changed`) has
    not moved beyond a per-`device_class` limit (6 h default, 3 h for `voltage`,
    `frequency`, `power_factor`), skipping zero-valued electrical sensors.
  - `entities` attribute listing each frozen sensor with its age in hours.
  - *Aviso: entidades com valor parado* and *Limpar aviso de entidades com valor
    parado* automations for notification create/dismiss.
  - `group.ignored_frozen_entities` and `ignored_from_frozen` label for exclusions.
  - The detection logic is kept in a single place: the `entities` attribute is
    the source of truth and the sensor `state` is derived from its length, so the
    count and the list can never drift apart.
- `LICENSE` (GNU GPL v3.0), `README.md`, `.gitignore`, and this changelog.

[1.2.0]: https://github.com/fapgomes/ha-package-entity-health/releases/tag/v1.2.0
[1.1.1]: https://github.com/fapgomes/ha-package-entity-health/releases/tag/v1.1.1
[1.1.0]: https://github.com/fapgomes/ha-package-entity-health/releases/tag/v1.1.0
[1.0.0]: https://github.com/fapgomes/ha-package-entity-health/releases/tag/v1.0.0
