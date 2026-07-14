# JC8012P4A1 Home Assistant Dashboard

A custom Home Assistant wall-panel dashboard for the **JC8012P4A1** board
(ESP32-P4, 10.1" 800×1280 capacitive touchscreen), built with
[ESPHome](https://esphome.io) and LVGL. Displayed in landscape
(1280×800), with a light "iOS card" style UI.

![status](https://img.shields.io/badge/ESPHome-2026.6%2B-blue)
![board](https://img.shields.io/badge/board-JC8012P4A1%20(ESP32--P4)-informational)

## Features

- **Clock & weather** — large clock, outdoor temperature, humidity and
  sunset time, all bound to Home Assistant entities.
- **Trash calendar (Müllkalender)** — up to 5 waste-collection fractions
  with days-remaining countdowns.
- **Kitchen media player** — transport controls (previous / play-pause /
  stop / next), a large touch-friendly volume slider, and 3 one-tap
  internet-radio quick-select buttons.
- **Light scenes** — momentary trigger buttons that fire an HA scene and
  reset themselves (no stuck toggle state).
- **Energy** — today's consumption and solar generation.
- **Ambient screensaver** — dims the backlight and shows a large clock
  after an idle timeout, driven by a custom touch-based timer (not
  LVGL's built-in idle detection, which proved unreliable on this
  hardware).
- **Browser-based layout planner** (`tools/dashboard_planner.html`) — a
  self-contained HTML tool for visually planning the dashboard layout:
  drag boxes and individual widgets (labels, buttons, icons, switches,
  sliders) around a 1:1 scale mockup of the screen, edit text content,
  alignment, font sizes and spacing, then export a YAML layout spec to
  use as a reference when editing `dashboard.yaml`. Open the file
  directly in any modern browser — no build step, no server needed.

## Hardware

- ESP32-P4 main SoC + ESP32-C6 co-processor (Wi-Fi over SDIO via
  `esp32_hosted`)
- `jd9365` MIPI-DSI display driver and `gsl3680` capacitive touch
  controller — vendored locally under `components/` and patched for
  current ESPHome versions, since upstream support for this exact panel
  is not mainlined.

## Repository layout

```
dashboard.yaml     – main ESPHome config (hardware + LVGL UI)
entities.yaml       – Home Assistant entity bindings — ADAPT THIS to your setup
secrets.yaml.example – template for Wi-Fi/OTA/API credentials
assets/             – fonts and icon glyph definitions
theme/              – LVGL color palette and widget theme
components/         – vendored jd9365 (display) and gsl3680 (touch) drivers
spares/             – optional extra hardware config snippets (buttons, sdcard,
                      ethernet, speaker, voice) not enabled by default
tools/dashboard_planner.html – standalone browser layout-planning tool
```

## Setup

1. Install [ESPHome](https://esphome.io/guides/installing_esphome) (2026.6 or
   newer — earlier versions can hit a boot-time memory-exhaustion crash on
   this board/UI combination).
2. Copy `secrets.yaml.example` to `secrets.yaml` and fill in your own Wi-Fi
   credentials, static IP, and OTA password. **Never commit `secrets.yaml`** —
   it's already gitignored.
3. Edit `entities.yaml` and replace every entity ID with the equivalents from
   your own Home Assistant instance (media player, trash sensors, scenes,
   energy/solar sensors, outdoor temperature, sunset).
4. Compile and flash:
   ```
   esphome compile dashboard.yaml
   esphome upload dashboard.yaml   # first flash must be via USB (see note below)
   ```

### A note on flashing

OTA uploads (`esphome upload` over Wi-Fi) can trigger ESPHome's automatic
safe-mode rollback on this board if a boot is slow or flaky, silently
reverting to the previous firmware. The reliable method is a wired USB flash
via `esptool`:

```
esptool --port /dev/tty.<your-port> write-flash 0x0 \
  .esphome/build/<name>/.pioenvs/<name>/firmware.factory.bin
```

Always verify the device boots cleanly (check the serial log for a single
boot entry, no `assert failed` / `Guru Meditation` panics) before relying on
OTA for subsequent updates.

## The layout planner

Open `tools/dashboard_planner.html` directly in a browser (double-click the
file, or `open tools/dashboard_planner.html`). It renders a 1:1 mockup of the
1280×800 display with every real widget as an individually selectable,
draggable, resizable element:

- Click a **box** (card) to move/resize it as a whole.
- Click a **single element** inside a box (a label, button, icon circle,
  switch, or slider) to reposition/resize just that widget, edit its text
  (double-click to edit inline, or use the inspector), change its font size,
  or pick an accent color.
- Add or remove elements per box from the inspector panel.
- **Export → YAML** produces a layout spec (position/size/text/alignment/
  font size per element) you can use as a reference when translating changes
  back into `dashboard.yaml`. It is a planning aid, not a file ESPHome
  consumes directly.

## License

See [LICENSE](LICENSE).
