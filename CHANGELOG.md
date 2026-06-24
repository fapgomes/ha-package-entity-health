# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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

[1.0.0]: https://github.com/fapgomes/ha-package-entity-health/releases/tag/v1.0.0
