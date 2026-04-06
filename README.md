# esphome-modular-lvgl-buttons

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![ESPHome](https://img.shields.io/badge/ESPHome-2025.1+-blue)](https://esphome.io)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Integration-41BDF5)](https://www.home-assistant.io/)

A modular component library for building touchscreen smart home control panels using [ESPHome](https://esphome.io/) + [LVGL](https://lvgl.io/) on cheap ESP32 displays.

This is a fork of [agillis/esphome-modular-lvgl-buttons](https://github.com/agillis/esphome-modular-lvgl-buttons) with a redesigned component architecture. See [ARCHITECTURE.md](ARCHITECTURE.md) for the full design rationale.

---

## How it works

Each entity type lives in `ui/<type>/` and provides three files:

```
ui/<type>/local.yaml    — tile for an ESPHome component on the same device
ui/<type>/remote.yaml   — tile for a Home Assistant entity
ui/<type>/detail.yaml   — full-screen detail page (complex types only)
```

Your device YAML composes a panel by including one hardware file, the shared infrastructure, and one `!include` per tile:

```yaml
packages:
  hardware: !include esphome-modular-lvgl-buttons/hardware/waveshare-esp32-s3-touch-lcd-4-v4.yaml
  colors:   !include esphome-modular-lvgl-buttons/common/color.yaml
  fonts:    !include esphome-modular-lvgl-buttons/common/fonts.yaml
  glyphs:   !include esphome-modular-lvgl-buttons/common/mdi_glyph_substitutions.yaml
  theme:    !include esphome-modular-lvgl-buttons/common/theme_style.yaml
  wifi:     !include esphome-modular-lvgl-buttons/common/wifi.yaml

  kitchen_light: !include
    file: esphome-modular-lvgl-buttons/ui/light/remote.yaml
    vars:
      uid: kitchen_light
      entity_id: "light.kitchen"
      row: 0
      column: 0
      text: "Kitchen"
      icon: $mdi_ceiling_light
```

---

## Available entity types

| Type | local | remote | Detail page | Notes |
|---|---|---|---|---|
| `light` | ✅ | ✅ | ✅ | RGB / CCT / brightness, capability auto-detected |
| `switch` | ✅ | ✅ | — | Works with any toggleable HA entity |
| `sensor` | ✅ | ✅ | — | Configurable unit and decimal precision |
| `climate` | ✅ | ✅ | ✅ | Arc setpoint, mode + fan dropdowns, capability flags |
| `cover` | 🔜 | 🔜 | 🔜 | Blinds, shutters, garage doors |
| `fan` | 🔜 | 🔜 | 🔜 | — |
| `button` | ✅ | ✅ | — | Momentary press — works with `script.*`, `scene.*` too |
| `number` | 🔜 | 🔜 | 🔜 | Setpoints, PID targets |
| `select` | 🔜 | 🔜 | 🔜 | Operating modes, option lists |
| `lock` | 🔜 | 🔜 | 🔜 | With PIN pad detail page |
| `alarm_control_panel` | 🔜 | 🔜 | 🔜 | Arm/disarm with PIN pad |
| `media_player` | 🔜 | 🔜 | 🔜 | — |
| `binary_sensor` | ✅ | ✅ | — | Read-only — door, motion, leak |
| `text_sensor` | ✅ | ✅ | — | Display any string state or attribute |

See each type's `README.md` for full variable reference and usage examples:

- [ui/light/README.md](ui/light/README.md)
- [ui/switch/README.md](ui/switch/README.md)
- [ui/sensor/README.md](ui/sensor/README.md)
- [ui/binary_sensor/README.md](ui/binary_sensor/README.md)
- [ui/text_sensor/README.md](ui/text_sensor/README.md)
- [ui/climate/README.md](ui/climate/README.md)
- [ui/button/README.md](ui/button/README.md)

---

## Quick start

### 1. Prerequisites

ESPHome 2025.1 or later. For SVG image support:

```bash
pip install cairosvg
```

### 2. Clone into your ESPHome config directory

```bash
cd /config   # or wherever your ESPHome configs live
git clone https://github.com/iezhkv/esphome-modular-lvgl-buttons.git
```

### 3. Set up secrets

Create `secrets.yaml` in your ESPHome config root (one level above this repo — never inside it):

```yaml
wifi_ssid: "your-ssid"
wifi_password: "your-wifi-password"
ap_password: "your-fallback-ap-password"
latitude: 0.0000
longitude: 0.0000
api_encryption_key: "your-base64-key"   # generate: python3 -c "import secrets,base64; print(base64.b64encode(secrets.token_bytes(32)).decode())"
ota_password: "your-ota-password"
```

### 4. Create your device config

Start from the template below or copy the closest example from `example_code/`.

```yaml
substitutions:
  icon_font: mdi_icons_40
  text_font: nunito_20
  button_on_color:  "ep_orange"
  button_off_color: "very_dark_gray"
  icon_on_color:    "yellow"
  icon_off_color:   "gray"
  label_on_color:   "white"
  label_off_color:  "gray"
  display_daytime_brightness: "100%"
  display_nighttime_brightness: "50%"
  display_night_hour: "22"
  display_night_minute: "0"
  display_backlight_timeout_always_enabled: "false"
  display_backlight_timeout_initial: "30"
  screen_width: "480"
  screen_height: "480"

esphome:
  name: my-panel
  friendly_name: My Panel
  on_boot:
  - priority: 400
    then:
    - script.execute: update_loading_page

logger:

script:
- id: time_update
  then:
  - lambda: return;

packages:
  wifi:           !include esphome-modular-lvgl-buttons/common/wifi.yaml
  ota_screen:     !include esphome-modular-lvgl-buttons/common/ota.yaml
  colors:         !include esphome-modular-lvgl-buttons/common/color.yaml
  fonts:          !include esphome-modular-lvgl-buttons/common/fonts.yaml
  glyphs:         !include esphome-modular-lvgl-buttons/common/mdi_glyph_substitutions.yaml
  sensors:        !include esphome-modular-lvgl-buttons/common/sensors_base.yaml
  theme_style:    !include esphome-modular-lvgl-buttons/common/theme_style.yaml
  backlight:      !include esphome-modular-lvgl-buttons/common/backlight_time.yaml
  hardware:       !include esphome-modular-lvgl-buttons/hardware/<your-hardware>.yaml
  loading_screen: !include esphome-modular-lvgl-buttons/pages/loading.yaml
  info_screen:    !include esphome-modular-lvgl-buttons/pages/info.yaml

  my_light: !include
    file: esphome-modular-lvgl-buttons/ui/light/remote.yaml
    vars:
      uid: my_light
      entity_id: "light.living_room"
      row: 0
      column: 0
      text: "Living Room"
      icon: $mdi_ceiling_light

lvgl:
  buffer_size: 100%
  pages:
  - id: main_page
    layout: 2x3
    styles: page_style
    <<: !include esphome-modular-lvgl-buttons/common/swipe_navigation.yaml

font:
- file: 'https://github.com/Templarian/MaterialDesign-Webfont/raw/v7.4.47/fonts/materialdesignicons-webfont.ttf'
  id: mdi_icons_40
  size: 40
  bpp: 8
  glyphs:
  # required by light detail page
  - $mdi_brightness_6
  - $mdi_chevron_up
  - $mdi_circle_opacity
  - $mdi_eyedropper
  - $mdi_lightbulb
  # your tile icons
  - $mdi_ceiling_light
  - $mdi_information_box
```

### 5. Flash

```bash
esphome run my-panel.yaml
```

---

## Grid layout

Pages use LVGL's grid layout. The `layout: NxM` shorthand creates an N-row × M-column grid. Tiles are placed with `row` and `column` (0-based).

| Layout | Rows | Columns | Tiles |
|---|---|---|---|
| `1x1` | 1 | 1 | 1 |
| `2x2` | 2 | 2 | 4 |
| `2x3` | 2 | 3 | 6 |
| `3x3` | 3 | 3 | 9 |

---

## Desktop development with SDL

Test your UI on macOS or Linux without flashing hardware:

```bash
# macOS
brew install sdl2

# Ubuntu/Debian
sudo apt install libsdl2-dev
```

Use the SDL hardware config:

```yaml
hardware: !include esphome-modular-lvgl-buttons/hardware/SDL-lvgl.yaml
sensors:  !include esphome-modular-lvgl-buttons/common/sensors_base_sdl.yaml
```

Then `esphome run your-config.yaml` — a window opens simulating the display.

---

## Supported hardware

### Waveshare

| Model | Size | Resolution |
|---|---|---|
| `waveshare-esp32-s3-touch-lcd-4-v4` | 4.0" | 480×480 |
| `waveshare-esp32-s3-touch-lcd-4.3` | 4.3" | 800×480 |
| `waveshare-esp32-s3-touch-lcd-7` | 7.0" | 800×480 |
| `waveshare-esp32-s3-touch-lcd-7B` | 7.0" | 800×480 |
| `waveshare-esp32-s3-touch-lcd-2.8c` | 2.8" | 320×240 |
| `waveshare-esp32-s3-touch-lcd-3.5` | 3.5" | 480×320 |
| `waveshare-esp32-s3-touch-lcd-3.5b` | 3.5" | 320×480 |
| `waveshare-esp32-p4-wifi6-touch-lcd-4b` | 4.0" | 720×720 |
| `waveshare-esp32-p4-86-panel` | 4.0" | 720×720 |
| `waveshare-esp32-p4-wifi6-touch-lcd-7` | 7.0" | 1024×600 |
| `waveshare-esp32-p4-wifi6-touch-lcd-7b` | 7.0" | 1024×600 |
| `waveshare-esp32-p4-wifi6-touch-lcd-10.1` | 10.1" | 800×1280 |

### Guition

| Model | Size | Resolution |
|---|---|---|
| `guition-esp32-s3-4848s040` | 4.0" | 480×480 |
| `guition-esp32-jc4827w543` | 4.3" | 272×480 |
| `guition-esp32-jc8048w535` | 3.5" | 480×320 |
| `guition-esp32-jc8048w550` | 5.0" | 480×800 |
| `guition-esp32-p4-jc4880p443` | 4.3" | 480×800 |
| `guition-esp32-p4-jc8012p4a1` | 8.0" | 800×1280 |

### Sunton

| Model | Size | Resolution |
|---|---|---|
| `sunton-esp32-2432s028` | 2.8" | 320×240 |
| `sunton-esp32-2432s028R` | 2.8" | 320×240 |
| `sunton-esp32-4827s032R` | 3.2" | 480×320 |
| `sunton-esp32-8048s050` | 5.0" | 480×800 |
| `sunton-esp32-8048s070` | 7.0" | 480×800 |

### Other

| Model | Size | Resolution |
|---|---|---|
| `esp32-s3-box-3` | 2.4" | 320×240 |
| `lilygo-tdisplays3` | 1.9" | 170×320 |
| `elecrow-esp32-7inch` | 7.0" | 800×480 |
| `SDL-lvgl` | variable | desktop simulation |

---

## Upstream features

The following modules are present from the upstream fork and fully functional. They are not maintained by this fork — refer to the [upstream README](https://github.com/agillis/esphome-modular-lvgl-buttons) for documentation:

- `ui/weather/` — 4-day weather forecast via `weather.get_forecasts`
- `tides/` — NOAA tide data with gauge display
- `ui/solar/` — Enphase Envoy solar monitoring

---

## License

MIT — see [LICENSE](LICENSE).
