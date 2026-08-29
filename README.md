# Home Assistant Controller for M5Stack Core2

A physical Home Assistant remote built on the M5Stack Core2 and ESPHome. The
device is distributed as an ESPHome **package**: a Home Assistant user adds a
few substitutions to their own device config (which entities to control) and
points it at this repository, and the touchscreen/button UI is loaded in.

This project starts small — controlling Home Assistant lights — and is
structured so more pages (climate, media, timers, etc.) can be added the same
way over time.

## How it works

- [`core2.yaml`](core2.yaml) is the package entry point. It wires together the
  hardware, networking and UI packages below.
- [`src/main/hardware.yaml`](src/main/hardware.yaml) configures the Core2
  board: ESP32, PSRAM, the AXP192 power IC, SPI/I2C buses, the display,
  touchscreen and RTC.
- [`src/main/network.yaml`](src/main/network.yaml) configures Wi-Fi, the
  ESPHome API, OTA updates and the captive portal, using substitutions so
  credentials stay in the consuming device config.
- [`src/main/lights.yaml`](src/main/lights.yaml) tracks up to six Home
  Assistant light entities (state via `homeassistant` text sensors) and
  exposes a `light.toggle` script for each one.
- [`src/pages/lights.yaml`](src/pages/lights.yaml) draws the lights list on
  the screen and wires up the touchscreen and the three virtual buttons.

## Quick installation

1. In ESPHome, create a new device configuration and add your credentials as
   secrets.
2. Use the following configuration, replacing the entity IDs with your own
   Home Assistant light entities and the `url` with this repository.
3. Flash it to the Core2 over USB for the first install, then use OTA updates
   afterwards.

```yaml
substitutions:
  devicename: m5core2
  friendly_name: M5Core2

  wifi_ssid: !secret wifi_ssid
  wifi_password: !secret wifi_password
  api_encryption_key: !secret m5core2_api_encryption_key
  ota_password: !secret m5core2_ota_password

  light_entities:
    - name: "Living Room"
      entity_id: light.living_room
    - name: "Kitchen"
      entity_id: light.kitchen
    - name: "Bedroom"
      entity_id: light.bedroom

packages:
  core2_ha:
    url: https://github.com/<your-github-username>/m5stack-core2-home-assistant
    ref: main
    files:
      - core2.yaml
    refresh: 0s
```

Create the referenced secrets in ESPHome, for example:

```yaml
wifi_ssid: "your-wifi-network"
wifi_password: "your-wifi-password"
m5core2_api_encryption_key: "replace-with-an-ESPHome-api-key"
m5core2_ota_password: "replace-with-an-OTA-password"
```

`ref: main` follows the version currently published on `main`. `refresh: 0s`
makes ESPHome check the remote package on every configuration or build, which
is useful while tracking that branch but depends on GitHub being reachable
and can add build time. It is optional; pin a tag or commit in `ref` when
reproducible builds matter.

Only substitutions need to be set in the local device config; do not edit the
files under `src/`, those are the shared package.

## Lights

Configure up to six lights through the `light_entities` list. Each entry
requires a display `name` and Home Assistant `entity_id`; shorter lists leave
the remaining rows hidden.

Each configured light gets a row on the screen showing its name and current
on/off state (read live from Home Assistant). The selected light has a detail
view with an on/off control, four RGB color presets, and a 1500K-6500K color
temperature stepper. Color and temperature commands turn the light on.

## Navigation

The Core2's screen area is 320×240; below it is a capacitive touch strip
divided into three virtual buttons (there are no physical buttons on the
Core2).

| Control | Action |
| --- | --- |
| On-screen power tile | Turns the selected light on or off |
| On-screen color swatches | Applies an RGB color preset |
| On-screen kelvin stepper | Adjusts and applies color temperature |
| Button A (left) | Move selection to the previous light |
| Button B (middle) | Toggle the currently selected light |
| Button C (right) | Move selection to the next light |

## Hardware

The firmware targets the M5Stack Core2 and its ESP32 controller, using the
built-in 320×240 display, FT63x6 capacitive touch panel, AXP192 power
management IC and BM8563 RTC.

## Requirements

- An M5Stack Core2.
- Home Assistant with the ESPHome integration.
- ESPHome.
- Wi-Fi access for the Core2.
- The Home Assistant light entities you want to control.

## Project structure

```text
m5stack-core2-home-assistant/
├── core2.yaml                 # Package entry point
├── example-core2-config.yaml  # Example device config that uses the package
├── src/
│   ├── main/                  # Hardware, networking and light entity state
│   └── pages/                 # Screen/touchscreen/button UI logic
└── README.md
```

## Development

Create a local `secrets.yaml` for your credentials, install ESPHome, then
validate and compile locally against the example config:

```bash
python -m venv .venv
source .venv/bin/activate
pip install esphome
esphome config example-core2-config.yaml
esphome compile example-core2-config.yaml
```

## Roadmap

This project intentionally starts with lights only. Climate, media and other
pages can follow the same pattern: a `src/main/*.yaml` package for the
Home Assistant entity state, and a `src/pages/*.yaml` package for the screen
and input handling.
