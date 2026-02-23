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

### Alarm System
- MP3 alarm playback from SD card via I2S amplifier
- One-shot alarm — fires once at the set time, then auto-disables
- Adjustable volume (0–100%)
- Sound file selection from all MP3s found on the SD card
- Test playback to preview alarm sound at the current volume
- Configurable snooze duration (1–30 minutes) with a live countdown on screen
- Random inspirational wake-up messages displayed when the alarm fires

### Snooze
- **Tap** the encoder button while the alarm is ringing to snooze
- Display shows `Snooze: M:SS` with a live countdown
- When the countdown reaches zero, the alarm fires again
- Snooze indefinitely until you're ready to get up
- **Long press** (2 seconds) to fully dismiss the alarm

### On-Screen Menu (Rotary Encoder)
A full settings menu accessible by holding the encoder button for 2 seconds:

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
- **Alarm** and **Webpage** toggle instantly on press (no edit mode)
- **Test** toggles playback instantly on press
- **Long press** exits the menu from anywhere

### Web Interface
A responsive, dark-themed web interface accessible from any device on the same network:

- **Home page** — Set alarm time, toggle alarm on/off, select alarm sound, adjust volume, test playback, adjust display brightness
- **Manage Sound Files** — Upload new MP3 files to the SD card, view file list with sizes, delete unwanted files
- **Location Settings** — Search for your city using the Open-Meteo Geocoding API, set timezone automatically
- **Firmware Update** — OTA (Over-The-Air) firmware updates directly from the browser
- **Web access control** — Enable/disable the web interface from the on-screen menu to prevent unauthorized changes

### Captive Portal Setup (AP Mode)
- On first boot (or when no WiFi credentials are saved), the clock creates its own WiFi access point
- Connect to the AP and a captive portal opens automatically for WiFi configuration
- Hold the **BOOT button** (GPIO9) for 5 seconds at any time to force AP mode and reconfigure WiFi

### MP3 File Management
- Upload MP3 files to the SD card via the web interface with a progress bar
- Delete files you no longer need
- Files are automatically scanned on boot and after upload/delete
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

#### Other
| Signal | GPIO |
|--------|------|
| BOOT button | 9 (hold 5s → AP mode) |

---

## Software Dependencies

### Arduino Libraries

| Library | Purpose |
|---------|---------|
| [Arduino_GFX](https://github.com/moononournation/Arduino_GFX) | Display driver for ST7789 |
| [LVGL 8.4](https://github.com/lvgl/lvgl) | Graphics UI framework |
| [ESP8266Audio](https://github.com/earlephilhower/ESP8266Audio) | MP3 decoding and I2S audio output |
| [ESPAsyncWebServer](https://github.com/me-no-dev/ESPAsyncWebServer) | Async web server for config pages |
| [AsyncTCP](https://github.com/me-no-dev/AsyncTCP) | TCP library required by ESPAsyncWebServer |
| SD | SD card access (built-in) |
| SPI | SPI bus (built-in) |
| WiFi | WiFi connectivity (built-in) |
| DNSServer | Captive portal DNS (built-in) |
| Preferences | Persistent settings storage (built-in) |
| HTTPClient | Weather API requests (built-in) |
| Update | OTA firmware updates (built-in) |

### LVGL Configuration (`lv_conf.h`)

The following built-in fonts must be enabled:

```c
#define LV_FONT_MONTSERRAT_10  1   // Menu hint text
#define LV_FONT_MONTSERRAT_12  1   // Small UI text
#define LV_FONT_MONTSERRAT_14  1   // Menu items, weather, date
#define LV_FONT_MONTSERRAT_16  1   // Boot screens
#define LV_FONT_MONTSERRAT_20  1   // Date display, alarm label
#define LV_FONT_MONTSERRAT_24  1   // Weather temperature
```

A custom large clock font (`lv_font_clock_big.c`) is used for the time display.

### External APIs

| API | Purpose | Auth |
|-----|---------|------|
| [Open-Meteo Weather](https://open-meteo.com/) | Current weather + daily forecast | None (free) |
| [Open-Meteo Geocoding](https://open-meteo.com/en/docs/geocoding-api) | City search for location settings | None (free) |
| NTP (pool.ntp.org) | Time synchronization | None |

---

## Project Files

| File | Description |
|------|-------------|
| `BedsideClock.ino` | Main sketch — all logic in one file |
| `lv_conf.h` | LVGL configuration (place in your LVGL library folder) |
| `lv_font_clock_big.c` | Custom large clock font for LVGL |
| `weather_icons.h` | Weather icon declarations |
| `icon_*.c` | Pre-rendered Meteocons icon bitmaps |

---

## Setup

### 1. Hardware Assembly
Wire the components according to the pinout tables above. The EC11 encoder has 7 pins — connect all GND pins to ground. No external pull-up resistors are needed; the code uses internal pullups.

### 2. SD Card
- Format as **FAT32**
- Place `.mp3` files in the **root directory**
- Up to 32 files are supported

### 3. Arduino IDE Setup
1. Install the **ESP32** board package (select ESP32-C6)
2. Install all required libraries listed above
3. Place `lv_conf.h` in your LVGL library folder
4. Open `BedsideClock.ino` and upload

### 4. First Boot — WiFi Configuration
1. The clock starts in **AP mode** and creates a WiFi network
2. Connect to it with your phone or laptop
3. A captive portal opens — select your WiFi network and enter the password
4. The clock reboots, connects to WiFi, syncs time, and fetches weather

### 5. Configure Location
1. Navigate to the clock's IP address (shown at the bottom right of the display)
2. Click **Location Settings**
3. Search for your city — timezone is set automatically
4. Save and weather data will update immediately

---

## Usage

### Daily Use
The clock shows the time, date, current weather, and daily high/low temperatures. When an alarm is armed, it shows the alarm time in red below the date.

### Setting an Alarm
**Via the menu:** Hold the encoder button for 2 seconds, navigate to *Wake up*, press to edit, set the time, then navigate to *Alarm* and press to arm it.

**Via the web:** Open the clock's IP in a browser, set the time with the hour/minute dropdowns, and toggle the switch to arm.

### When the Alarm Fires
1. A random inspirational message appears in warm gold
2. The selected MP3 plays on loop
3. **Tap** the encoder to snooze — the display shows a countdown
4. **Hold** the encoder for 2 seconds to fully dismiss
5. The alarm auto-disables after firing (re-arm for the next day)

### Managing Sound Files
Navigate to the clock's IP → **Manage Sound Files** to upload new MP3s or delete existing ones. The currently active alarm sound is highlighted in gold.

### Re-entering WiFi Setup
Hold the **BOOT button** (GPIO9) for 5 seconds — a progress bar appears at the bottom of the screen. The clock restarts in AP mode for reconfiguration.

---

## Inspirational Wake-Up Messages

When the alarm fires, one of these messages is randomly displayed:

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

---

## License

MIT License — feel free to use, modify, and share.

---

## Credits

Built with love as a bedside clock for the family.

**KL.Design**
