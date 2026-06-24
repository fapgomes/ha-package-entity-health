# Entity Health — Home Assistant Package

A single Home Assistant [package](https://www.home-assistant.io/docs/configuration/packages/) that bundles two complementary entity‑health monitors:

| Monitor | Version | What it detects |
| --- | --- | --- |
| **Unavailable Entities** | v2.4 | Entities whose state is `unknown` / `unavailable`. |
| **Frozen (stuck‑value) Entities** | v1.1 | Sensors that are still *available* but whose **value** hasn't changed for too long. |

The Unavailable Entities part is based on [jazzyisj/unavailable-entities-sensor](https://github.com/jazzyisj/unavailable-entities-sensor). The Frozen Entities part is original to this package.

## Requirements

- Home Assistant **2024.8** or newer.
- `jq` available on the host (used by the `command_line` sensor that lists disabled device entities).

## Why two monitors?

An entity can be broken in two different ways:

1. **It goes `unavailable`** — the integration lost it. → *Unavailable Entities*.
2. **It stays available but stops updating its value** — the worst kind, because nothing looks wrong at a glance. → *Frozen Entities*.

### Why `last_changed` and not `last_reported` for frozen detection?

Some integrations (e.g. Zigbee2MQTT) keep **re‑publishing old values**, so `last_reported` keeps advancing even when the underlying device is stuck. For example, the Aqara sensor on the pool pump re‑publishes its topic whenever the temperature changes, so `last_reported` moves even if the sensor's own value is frozen. Only the stagnation of the **value itself** (`last_changed`) reveals the fault.

## Installation

1. Copy `package_entity_health.yaml` into your Home Assistant `packages/` directory (e.g. `config/packages/`).
2. Make sure packages are enabled in `configuration.yaml`:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

3. Restart Home Assistant (or reload YAML where supported).

## Entities provided

| Entity | Type | Purpose |
| --- | --- | --- |
| `sensor.disabled_device_entities` | command_line | Count + list of disabled device entities (read from `core.entity_registry`). |
| `sensor.unavailable_entities` | template | Count of unavailable entities (mirrors `group.unavailable_entities`). |
| `sensor.frozen_entities` | template | Count of frozen sensors. `entities` attribute lists each one with its age in hours; `entity_ids` attribute holds the raw ids (used to build the group). |
| `group.unavailable_entities` | group | Live group of unavailable entities, rebuilt once per minute. |
| `group.frozen_entities` | group | Live group of frozen sensors (raw entity_ids), rebuilt once per minute — use this in cards / `auto-entities`. |
| `group.ignored_entities` | group | Entities to exclude from *unavailable* detection. |
| `group.ignored_frozen_entities` | group | Entities to exclude from *frozen* detection. |

### Automations

- **Update Unavailable Entities Group** — rebuilds `group.unavailable_entities` every minute (and on `group.reload`).
- **Update Frozen Entities Group** — rebuilds `group.frozen_entities` every minute (and on `group.reload`) from `sensor.frozen_entities`.
- **Unavailable Entities Notification** — creates/dismisses a persistent notification based on the count.
- **Frozen Entities Alert** — notifies when frozen entities are detected for the configured delay (default 15 min): always a persistent notification, plus the notify service configured in `input_text.frozen_notify_service`. (Notification text is in Portuguese.)
- **Clear Frozen Entities Alert** — dismisses the frozen notification once the count returns to zero (after 5 min).

## How it works — timing

### Unavailable

`group.unavailable_entities` is rebuilt **once per minute**, so an entity that goes `unknown`/`unavailable` shows up within ~1 minute. Entities are only counted after they have been unavailable for at least 60 s (`ignore_seconds`), to avoid flapping during restarts.

### Frozen

There are three timing layers, so a frozen sensor doesn't appear instantly — that's by design:

1. **Detection threshold** — measured from the last time the *value* changed (`last_changed`):
   - **General limit:** 12 h by default (`frozen_default_limit_hours`).
   - **Tight limit:** 10 h by default (`frozen_tight_limit_hours`) for `voltage`, `frequency`, `power_factor` — these always vary on a live meter, so a shorter window is enough.
   - Electrical sensors (`power`, `current`, `energy`, `apparent_power`, `reactive_power`) reading `0` are **ignored** (the appliance is simply off).
   - Only `measurement`, `total`, `total_increasing` state classes are watched.
2. **`sensor.frozen_entities` / list** — updates within seconds of crossing the threshold (it re-renders whenever any `sensor.*` changes).
3. **`group.frozen_entities` / cards** — rebuilt **once per minute**, so up to ~1 min behind the list.
4. **Notification** — fires only after the count has been `> 0` for `frozen_notify_delay_minutes` straight (default 15 min).

So for a normal sensor: ~**12 h** stuck → in the list within seconds, in the group/card within ~1 min, notification ~15 min later. For the electrical classes it's **10 h** instead of 12 h.

> **`group.frozen_entities` shows `unknown` when there is nothing frozen** — that's the normal, healthy state of an empty group (same as `group.unavailable_entities` when nothing is unavailable). It populates as soon as a sensor crosses the threshold. Use the count `sensor.frozen_entities` (`0` when healthy) for display.

## Configuration

All tunable parameters are **input helpers defined at the top of `package_entity_health.yaml`**. Edit the `initial:` value and restart Home Assistant (you can also nudge them from the UI, but they reset to the `initial:` value on every restart, so the YAML stays the source of truth):

| Helper | Default | Purpose |
| --- | --- | --- |
| `input_number.frozen_default_limit_hours` | `12` | General "stuck for too long" threshold, in hours. |
| `input_number.frozen_tight_limit_hours` | `10` | Tighter threshold for `voltage` / `frequency` / `power_factor`. |
| `input_number.frozen_notify_delay_minutes` | `15` | Minutes the count must stay `> 0` before notifying. |
| `input_text.frozen_notify_service` | `notify.calvin` | Notify service for the external alert. **Leave empty to send only the persistent notification.** Change it to your own service (e.g. `notify.mobile_app_xxx`). |

The non‑tunable structural lists (`zero_skip`, watched state classes) stay inline in `sensor.frozen_entities`.

### Ignoring entities

**Unavailable** — add the entity to `group.ignored_entities`, apply the `ignored_from_unavailable` label, or apply it to a device. Many noisy entity patterns (robot‑vacuum config selects, kiosk/tablet sensors, Frigate, etc.) are already filtered by `rejectattr` rules in the update automation.

**Frozen** — add the entity to `group.ignored_frozen_entities` **or** apply the `ignored_from_frozen` label.

## Credits

- Unavailable Entities sensor by [Jason Nader (jazzyisj)](https://github.com/jazzyisj/unavailable-entities-sensor).
- Frozen Entities monitor and package assembly by [@fapgomes](https://github.com/fapgomes).

## License

Released under the [GNU General Public License v3.0](LICENSE).
