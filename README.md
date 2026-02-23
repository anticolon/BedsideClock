# ESP32-C6 Bedside Clock

A feature-rich bedside clock built on the ESP32-C6 with a Waveshare 1.47" LCD display, live weather, MP3 alarm with snooze, and a rotary encoder menu — all configurable via a built-in web interface.

![ESP32-C6](https://img.shields.io/badge/ESP32--C6-RISC--V-blue)
![Arduino](https://img.shields.io/badge/Arduino-IDE-teal)
![LVGL](https://img.shields.io/badge/LVGL-8.4-green)
![License](https://img.shields.io/badge/license-MIT-orange)

---

## Features

### Clock & Weather
- Large custom font clock display with automatic NTP time sync
- Automatic timezone and DST handling via Open-Meteo Geocoding API
- Live weather with pre-rendered Meteocons icons (sunny, cloudy, rain, snow, fog, thunderstorm, etc.)
- Current temperature with daily high/low display
- Weather refreshes every 15 minutes
- Balanced layout: clock and date on the left 3/4, weather panel on the right 1/4

### Alarm System
- MP3 alarm playback from SD card via I2S amplifier
- Continuous looping of the alarm sound until snoozed or dismissed
- One-shot alarm — fires once at the set time, then auto-disables (re-arm for the next day)
- Adjustable volume (0–100%) with live preview
- Sound file selection from all MP3s found on the SD card
- Test playback to preview the alarm sound at the current volume
- Configurable snooze duration (1–30 minutes) with a live countdown on screen
- Random inspirational wake-up messages displayed when the alarm fires

### Snooze
- **Tap** the encoder button while the alarm is ringing to snooze
- Display shows `Snooze: M:SS` in cyan with a live countdown
- When the countdown reaches zero, the alarm fires again with looping audio
- Snooze indefinitely until you're ready to get up
- **Long press** (1 second) to fully dismiss the alarm
- Menu is blocked during alarm and snooze states to prevent accidental changes

### Color-Coded Display States
| State | Color | Display |
|-------|-------|---------|
| Alarm armed | Red | `Alarm: 07:00` |
| Alarm firing | Warm gold | Random inspirational message |
| Snooze countdown | Cyan | `Snooze: 4:32` |

### On-Screen Menu (Rotary Encoder)
A full settings menu accessible by holding the encoder button for 1 second:

```
  Brightness:              75%
  Wake up:               07:00
  Alarm:                    On
  Snooze:                5 min
  Tune:             alarm.mp3
  Volume:                  50%
  Test:                  Play
  Webpage:                  On
```

- **Rotate** to navigate between items
- **Press** to enter edit mode (value turns green), rotate to adjust, press to confirm
- **Alarm** and **Webpage** toggle instantly on press (no edit mode needed)
- **Test** toggles sound playback instantly on press
- **Wake up** has two-stage editing: press to edit hour `[07]:00`, press again for minute `07:[00]`, press to save
- **Long press** (1 second) exits the menu from anywhere
- Long filenames are truncated with `...` to prevent overflow
- All settings are persisted to flash memory and survive reboots

### Web Interface
A responsive, dark-themed web interface accessible from any device on the same network:

- **Home page** — Set alarm time (24h format dropdowns), toggle alarm on/off, select alarm sound, adjust volume with slider, test playback, adjust display brightness
- **Manage Sound Files** — Upload new MP3 files to the SD card with a progress bar, view file list with file sizes, delete unwanted files, see SD card usage statistics
- **Location Settings** — Search for your city using the Open-Meteo Geocoding API, timezone is set automatically
- **Firmware Update** — OTA (Over-The-Air) firmware updates directly from the browser
- **Web access control** — Enable/disable the entire web interface from the on-screen menu to prevent unauthorized access. When disabled, all endpoints return `403 Forbidden`. OTA updates and the AP config portal remain accessible for recovery.

### Captive Portal Setup (AP Mode)
- On first boot (or when no WiFi credentials are saved), the clock creates its own WiFi access point
- Connect to the AP and a captive portal opens automatically for WiFi configuration
- Supports WiFi network scanning with signal strength display
- Hold the **BOOT button** (GPIO9) for 5 seconds at any time to force AP mode and reconfigure WiFi — a progress bar on screen shows the hold duration

### MP3 File Management
- Upload MP3 files to the SD card via the web interface with a real-time progress bar
- Delete files directly from the file manager page
- Currently active alarm sound is highlighted in gold
- SD card total and used space displayed at the bottom of the file manager
- Files are automatically rescanned on boot and after every upload or delete
- If the active alarm sound is deleted, the alarm sound setting is automatically cleared
- Supports up to 32 MP3 files in the root directory of the SD card

---

## Hardware

### Components
| Component | Description |
|-----------|-------------|
| ESP32-C6 | Microcontroller (RISC-V, WiFi 6, BLE 5) |
| Waveshare 1.47" LCD | 320×172 pixels, ST7789 driver, SPI interface |
| MAX98357 | I2S mono amplifier for audio output |
| EC11 Rotary Encoder | 7-pin, with push button for navigation |
| SD Card | Onboard (shares SPI with display), FAT32 formatted |
| Speaker | Connected to MAX98357 output terminals |

### Pinout

#### Display (ST7789 SPI)
| Signal | GPIO |
|--------|------|
| DC | 15 |
| CS | 14 |
| SCK | 7 |
| MOSI | 6 |
| RST | 21 |
| Backlight | 22 |

#### SD Card (shared SPI bus)
| Signal | GPIO |
|--------|------|
| MISO | 5 |
| CS | 4 |
| SCK | 7 (shared) |
| MOSI | 6 (shared) |

#### MAX98357 I2S Amplifier
| Signal | GPIO |
|--------|------|
| BCLK | 2 |
| LRC (WSEL) | 3 |
| DIN | 1 |
| VIN | 3.3V or 5V |
| GND | GND |

#### EC11 Rotary Encoder
| Signal | GPIO |
|--------|------|
| CLK (A) | 18 |
| DT (B) | 23 |
| SW (button) | 0 |
| GND (encoder) | GND |
| GND (switch) | GND |

No external pull-up resistors are needed — the code enables internal pullups on all encoder pins.

#### Other
| Signal | GPIO |
|--------|------|
| BOOT button | 9 (hold 5s → AP mode) |

---

## Software Dependencies

### Arduino Libraries

| Library | Purpose | License |
|---------|---------|---------|
| [Arduino_GFX](https://github.com/moononournation/Arduino_GFX) | Display driver for ST7789 | BSD |
| [LVGL 8.4](https://github.com/lvgl/lvgl) | Graphics UI framework | MIT |
| [ESP8266Audio](https://github.com/earlephilhower/ESP8266Audio) | MP3 decoding and I2S audio output | GPL-3.0 |
| [ESPAsyncWebServer](https://github.com/me-no-dev/ESPAsyncWebServer) | Async web server for config pages and file uploads | LGPL-2.1 |
| [AsyncTCP](https://github.com/me-no-dev/AsyncTCP) | TCP library required by ESPAsyncWebServer | LGPL-3.0 |
| SD | SD card access | Built-in |
| SPI | SPI bus | Built-in |
| WiFi | WiFi connectivity | Built-in |
| DNSServer | Captive portal DNS | Built-in |
| Preferences | Persistent settings storage (NVS) | Built-in |
| HTTPClient | Weather API requests | Built-in |
| Update | OTA firmware updates | Built-in |

### LVGL Configuration (`lv_conf.h`)

The following built-in Montserrat fonts must be enabled:

```c
#define LV_FONT_MONTSERRAT_10  1   // Menu hint text
#define LV_FONT_MONTSERRAT_12  1   // Small UI text, IP address
#define LV_FONT_MONTSERRAT_14  1   // Menu items, weather hi/lo, date
#define LV_FONT_MONTSERRAT_16  1   // Boot screen status
#define LV_FONT_MONTSERRAT_20  1   // Date display, alarm/snooze label
#define LV_FONT_MONTSERRAT_24  1   // Weather current temperature
```

A custom large clock font (`lv_font_clock_big.c`) is used for the main time display.

### External APIs

| API | Purpose | Auth |
|-----|---------|------|
| [Open-Meteo Weather](https://open-meteo.com/) | Current weather + daily high/low forecast | None (free, no key required) |
| [Open-Meteo Geocoding](https://open-meteo.com/en/docs/geocoding-api) | City search for automatic timezone/location | None (free, no key required) |
| NTP (pool.ntp.org) | Time synchronization | None |

---

## Project Files

| File | Description |
|------|-------------|
| `BedsideClock.ino` | Main sketch — all logic in a single file |
| `lv_conf.h` | LVGL configuration (place in your LVGL library folder) |
| `lv_font_clock_big.c` | Custom large clock font for LVGL |
| `weather_icons.h` | Weather icon declarations |
| `icon_*.c` | Pre-rendered Meteocons weather icon bitmaps |

---

## Setup

### 1. Hardware Assembly
Wire the components according to the pinout tables above. The EC11 encoder typically has 7 pins — two are for the rotary (CLK, DT), one for the push button (SW), one for VCC (+), and the remaining are GND. Connect all GND pins to ground.

### 2. SD Card
- Format as **FAT32**
- Place `.mp3` files in the **root directory**
- Up to 32 files are supported
- Files can also be uploaded later via the web interface

### 3. Arduino IDE Setup
1. Install the **ESP32** board package (select ESP32-C6 as your board)
2. Install all required external libraries listed above
3. Place `lv_conf.h` in your LVGL library folder (typically `libraries/lvgl/`)
4. Open `BedsideClock.ino` and upload

### 4. First Boot — WiFi Configuration
1. The clock starts in **AP mode** and creates a WiFi access point
2. Connect to it with your phone or laptop
3. A captive portal opens automatically — select your WiFi network and enter the password
4. The clock reboots, connects to WiFi, syncs time via NTP, and fetches weather

### 5. Configure Location
1. Navigate to the clock's IP address (shown at the bottom right of the display)
2. Click **Location Settings**
3. Search for your city — timezone and coordinates are set automatically
4. Save — weather data updates immediately

---

## Usage

### Daily Use
The clock displays the time, date, current weather icon, temperature, and daily high/low. When an alarm is armed, the set time appears in red below the date. The clock's IP address is shown in the bottom right corner for easy web access.

### Setting an Alarm
**Via the on-screen menu:** Hold the encoder button for 1 second to open the menu. Navigate to *Wake up* and press to edit the time. Then navigate to *Alarm* and press to toggle it on.

**Via the web interface:** Open the clock's IP in a browser, set the hour and minute with the dropdown selectors, and toggle the switch to arm. You can also select the alarm sound, adjust volume, and test playback.

### When the Alarm Fires
1. A random inspirational message appears in warm gold text
2. The selected MP3 plays continuously on loop
3. **Tap** the encoder to snooze — the display shows a cyan countdown timer
4. When the snooze timer expires, the alarm fires again
5. **Long press** the encoder (1 second) to fully dismiss the alarm
7. The alarm auto-disables after firing — re-arm it for the next day

### Managing Sound Files
Navigate to the clock's IP → **Manage Sound Files** to:
- **Upload** new MP3 files with a visual progress bar
- **Delete** files you no longer want
- See file sizes and total SD card usage
- The currently active alarm sound is highlighted in gold

### Disabling the Web Interface
For security, you can disable the web interface from the on-screen menu. Navigate to *Webpage* and press to toggle it off. When disabled, all web requests return `403 Forbidden`. The setting persists across reboots. OTA firmware updates and the AP captive portal are not affected, so you can always recover access.

### Re-entering WiFi Setup
Hold the **BOOT button** (GPIO9) for 5 seconds — a progress bar appears at the bottom of the screen showing the hold duration. The clock restarts in AP mode for WiFi reconfiguration.

---

## Inspirational Wake-Up Messages

When the alarm fires, one of these messages is randomly chosen and displayed in warm gold:

- *Have a wonderful day!*
- *This day is full of potential!*
- *Dad loves you!*
- *Mom loves you!*
- *You are beautiful!*
- *You make the world brighter!*
- *Today is going to be amazing!*
- *You are so loved!*
- *The world needs your smile!*
- *You are stronger than you know!*
- *Great things await you today!*
- *You are enough, always!*
- *Today is yours to shine!*
- *Be proud of who you are!*
- *Your family believes in you!*

The same message persists through snooze cycles — a fresh random pick happens the next time the alarm triggers.

---

## Licenses & Attributions

### Project License
This project is released under the **MIT License** — feel free to use, modify, and share.

### Fonts
- **Montserrat** — Used for all UI text via LVGL built-in fonts. Designed by Julieta Ulanovsky, Sol Matas, Juan Pablo del Peral, and Jacques Le Bailly. Licensed under the [SIL Open Font License 1.1](https://scripts.sil.org/OFL).
- **Clock display font** — Custom large font (`lv_font_clock_big.c`) generated for LVGL from Montserrat, also under the SIL Open Font License 1.1.

### Weather Icons
- **Meteocons** by Bas Milius — Beautiful weather icons used for the weather display. Pre-rendered as C bitmap arrays for LVGL. Licensed under the [MIT License](https://github.com/basmilius/weather-icons/blob/dev/LICENSE). Source: [github.com/basmilius/weather-icons](https://github.com/basmilius/weather-icons)

### Libraries
- **LVGL** — MIT License
- **Arduino_GFX** — BSD License
- **ESP8266Audio** — GPL-3.0 License
- **ESPAsyncWebServer** — LGPL-2.1 License
- **AsyncTCP** — LGPL-3.0 License

### APIs
- **Open-Meteo** — Free weather and geocoding API, no API key required. [open-meteo.com](https://open-meteo.com/)

---

## Credits

Built with love as a bedside clock for the family.

**KL.Design**
