## Display (ST7735 0.96" "blue tab")

The 0.96" ST7735 IPS panel on the T-Dongle-S3 is a **"blue tab"**
variant (named after the colour of the screen-protector pull tab).
Getting a clean image in **landscape** orientation requires a
specific combination of parameters that is not obvious from the
component defaults. With default settings the panel typically shows
a garbled/noisy image with undriven borders, and colours are wrong
(e.g. red renders as yellow).

The following values produce a full, correctly-oriented,
correctly-coloured landscape (160x80) image:

| Parameter        | Value | Notes                                         |
| ---------------- | ----- | --------------------------------------------- |
| `device_width`   | 80    | native panel width (portrait)                 |
| `device_height`  | 160   | native panel height (portrait)                |
| `col_start`      | 26    | column offset to align the visible window     |
| `row_start`      | 1     | row offset to align the visible window        |
| `rotation`       | 90    | landscape (use 270 for the opposite side)     |
| `invert_colors`  | true  | required, otherwise colours are inverted      |
| `use_bgr`        | true  | required, otherwise red/blue channels swap    |
| `eight_bit_color`| true  | recommended for this panel                    |

Note: the `st7735` platform is deprecated in favour of `mipi_spi`,
but at the time of writing `mipi_spi` has a known bug where
`rotation: 180` ignores manual offsets on similar panels
(esphome/esphome#17050), so `st7735` remains the more reliable
choice for this display for now.

The RGB status LED is an **APA102**, driven over a separate SPI bus
(clk=GPIO39, mosi=GPIO40) using the `spi_led_strip` platform.

# ============================================================
# T-Dongle-S3 — Working display configuration (ST7735 blue-tab)
# Example config for devices.esphome.io
# ============================================================
# The 0.96" ST7735 panel on the T-Dongle-S3 is a "blue tab"
# variant. Getting a clean, correctly-oriented, correctly-
# coloured image in landscape requires a specific combination
# of parameters that is not obvious from defaults:
#
#   - device_width: 80 / device_height: 160 (native portrait)
#   - col_start: 26 / row_start: 1  (offsets to fill the panel
#     without noise/garbled borders)
#   - rotation: 90  (landscape)
#   - invert_colors: true + use_bgr: true  (correct colours;
#     without these, red renders as yellow/cyan)
# ============================================================

spi:
  clk_pin: GPIO5
  mosi_pin: GPIO3

display:
  - platform: st7735
    model: "INITR_MINI160X80"
    reset_pin: GPIO1
    cs_pin: GPIO4
    dc_pin: GPIO2
    rotation: 90
    device_width: 80
    device_height: 160
    col_start: 26
    row_start: 1
    eight_bit_color: true
    invert_colors: true
    use_bgr: true
    update_interval: 1s
    lambda: |-
      it.fill(Color::BLACK);
      it.print(6, 6, id(my_font), Color(0, 207, 255), "T-Dongle-S3");
      it.print(6, 30, id(my_font), Color(255, 255, 255), "Display OK");

font:
  - file: "gfonts://Montserrat@700"
    id: my_font
    size: 16

# Backlight (GPIO38, inverted)
output:
  - platform: ledc
    pin: GPIO38
    inverted: true
    id: backlight_output
    frequency: 2000

light:
  - platform: monochromatic
    output: backlight_output
    name: "LCD Backlight"
    restore_mode: ALWAYS_ON

# APA102 RGB status LED (separate SPI bus)
# clk=GPIO39, mosi=GPIO40
