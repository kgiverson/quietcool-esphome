# QuietCool ESPHome Session Handoff

Last updated: June 12, 2026

This document records the working state established while adapting this
repository for a three-speed QuietCool attic fan and an M5Stack ATOM Lite.

## Tested Hardware

- ESPHome bridge: M5Stack ATOM Lite
- ATOM USB serial port during setup: `/dev/ttyUSB0`
- ATOM MAC: `14:08:08:55:44:2c`
- QuietCool BLE name: `ATICFAN_ece334ed7fc2`
- QuietCool BLE MAC: `EC:E3:34:ED:7F:C2`
- QuietCool model reported over BLE: `AFG SMT ES-3.0`
- QuietCool fan type: `THREE`
- QuietCool serial reported over BLE: `GSE3043883`

The fan controller and ATOM Lite are paired and communicating. The ATOM holds
the BLE connection, so the QuietCool phone app generally cannot connect while
the ATOM is powered and connected.

## Files Changed

- `quietcool-smart-attic-fan-control.yaml`
  - Added newer numeric V2 protocol support.
  - Added robust parsing for `QQ`-prefixed and fragmented responses.
  - Added three-speed, mode, timer, and pairing controls.
- `quietcool-m5stack-atom-lite.yaml`
  - Local configuration for the tested ATOM Lite and fan.
- `README.md`
  - Added ATOM Lite setup and Home Assistant control documentation.
- `.github/workflows/ci.yml`
  - Removed a CI reference to a nonexistent factory YAML file.

These changes were not committed at the time this handoff was written.

## Confirmed V2 BLE Protocol

Newer controller firmware uses numeric API calls and short JSON keys:

| Operation | Request |
| --- | --- |
| Login | `{"A":13,"P":"pair_id"}` |
| Pair | `{"A":14,"P":"pair_id"}` |
| Get work state | `{"A":1}` |
| Get parameters | `{"A":2}` |
| Get fan information | `{"A":17}` |
| Set mode | `{"A":9,"M":"Idle|Run|Timer|TH"}` |
| Set continuous-run speed | `{"A":18,"S":"LOW|HIGH"}` |
| Set timer | `{"A":7,"H":hours,"M":minutes,"R":"LOW|MEDIUM|HIGH"}` |

Responses may arrive in multiple BLE notifications and may be prefixed with
`QQ`. The parser accumulates notifications, strips bytes before the first JSON
object, and parses the completed object.

### Modes

- `Idle`: fan off
- `Run`: continuous manual run
- `Timer`: timed run
- `TH`: smart temperature and humidity mode

Continuous `Run` speed has only been confirmed for `LOW` and `HIGH`.
`{"A":18,"S":"MEDIUM"}` was ignored by this controller. Medium works through
the timer command, so Manual Run Medium currently uses a `23h 59m` timer
fallback.

## Home Assistant Behavior

### QuietCool Fan

This fan entity is the sole on/off control:

- Turning it on starts the selected **Operating Mode**.
- Turning it off sends mode `Idle`.

Its percentage speed control also starts Manual Run at the selected speed.

### Operating Mode

The selector contains:

- Manual Run
- Timer Run
- Smart Temperature & Humidity

There is intentionally no Off option. Changing Operating Mode while the fan is
off selects what the fan toggle will start next. Changing it while the fan is
on switches modes immediately.

### Manual Speed

- Low and High use continuous `Run` mode.
- Medium uses `Timer` mode with a `23h 59m` duration because continuous Medium
  is not accepted by this controller.
- Re-select Medium to restart that long timer if needed.

### Timer Run

Timer Run uses **Timer Hours**, **Timer Minutes**, and **Manual Speed**.

### Smart Temperature & Humidity

Smart mode sends mode `TH` and uses the active thresholds stored in the
QuietCool controller. Applying a preset in the QuietCool app appears to copy
that preset's values into these active thresholds; `TH` does not appear to
reference a named preset while running.

The tested controller reported these active values during the session:

- Temperature high: `100°F`
- Temperature medium: `90°F`
- Temperature low: `80°F`
- Humidity high: `90%`
- Humidity low: `70%`
- Humidity speed/range: `LOW`

The threshold sensors are disabled by default in Home Assistant. They can be
enabled from the ESPHome device's entity list.

### Pair State

`Pair State: No` is normal after pairing. It means the controller is not
currently accepting another pairing, not that the ATOM is unpaired.

## Build And Flash

Validate the ATOM configuration:

```sh
uvx --from esphome esphome config quietcool-m5stack-atom-lite.yaml
```

Build and flash over USB:

```sh
uvx --from esphome esphome run quietcool-m5stack-atom-lite.yaml \
  --device /dev/ttyUSB0 --no-logs
```

Read serial logs:

```sh
uvx --from esphome esphome logs quietcool-m5stack-atom-lite.yaml \
  --device /dev/ttyUSB0
```

The tested USB connection consistently fails its first upload attempt at
`460800` baud and succeeds automatically on the retry at `115200`.

## Current Limitations And Follow-Up Work

- The Home Assistant controls are optimistic. They issue commands but do not
  fully synchronize their displayed state from the controller's reported mode.
- Manual Medium is a long-timer fallback, not true continuous `Run`.
- The active Smart thresholds are readable but not currently editable from
  Home Assistant.
- The generic `temperature` and `humidity` entities remain unknown; live values
  are provided by `temp_sample` and `humidity_sample`.
- The local ATOM configuration contains the tested fan's BLE MAC. Review it
  before sharing or deploying to another controller.

Useful protocol research:

- <https://github.com/awkaplan/quietcool-esphome/pull/2>
- <https://github.com/emerose/quietcool>
- <https://github.com/snyamathi/quietcool>

