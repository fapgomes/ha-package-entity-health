# Entity Health — Home Assistant Package

A single Home Assistant [package](https://www.home-assistant.io/docs/configuration/packages/) that bundles two complementary entity‑health monitors:

| Monitor | Version | What it detects |
| --- | --- | --- |
| **Unavailable Entities** | v2.4 | Entities whose state is `unknown` / `unavailable`. |
| **Frozen (stuck‑value) Entities** | v1.0 | Sensors that are still *available* but whose **value** hasn't changed for too long. |

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
| `sensor.frozen_entities` | template | Count of frozen sensors, with an `entities` attribute listing each one and its age in hours. |
| `group.unavailable_entities` | group | Live group of unavailable entities, rebuilt once per minute. |
| `group.ignored_entities` | group | Entities to exclude from *unavailable* detection. |
| `group.ignored_frozen_entities` | group | Entities to exclude from *frozen* detection. |

### Automations

- **Update Unavailable Entities Group** — rebuilds `group.unavailable_entities` every minute (and on `group.reload`).
- **Unavailable Entities Notification** — creates/dismisses a persistent notification based on the count.
- **Aviso: entidades com valor parado** — notifies (persistent + `notify.calvin`) when frozen entities are detected for 15 min.
- **Limpar aviso de entidades com valor parado** — dismisses the frozen notification once the count returns to zero.

> **Note:** the frozen notification uses `notify.calvin`. Change this to your own notify service.

## Configuration

### Frozen detection thresholds

The detection logic lives in `sensor.frozen_entities`:

- **Default limit:** `21600` s (6 h) of no value change.
- **Tight limit:** `10800` s (3 h) for `voltage`, `frequency`, `power_factor` — these always vary on a live meter, so a short window is enough.
- **Zero‑skip:** sensors of class `power`, `current`, `energy`, `apparent_power`, `reactive_power` reading `0` are ignored (the appliance is simply off).
- **Watched state classes:** `measurement`, `total`, `total_increasing`.

### Ignoring entities

**Unavailable** — add the entity to `group.ignored_entities`, apply the `ignored_from_unavailable` label, or apply it to a device. Many noisy entity patterns (robot‑vacuum config selects, kiosk/tablet sensors, Frigate, etc.) are already filtered by `rejectattr` rules in the update automation.

**Frozen** — add the entity to `group.ignored_frozen_entities` **or** apply the `ignored_from_frozen` label.

## Credits

- Unavailable Entities sensor by [Jason Nader (jazzyisj)](https://github.com/jazzyisj/unavailable-entities-sensor).
- Frozen Entities monitor and package assembly by [@fapgomes](https://github.com/fapgomes).
