# KMB ETA Display

A desk display for [ESP32-2432S028R](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display) (Cheap Yellow Display / CYD) that shows Hong Kong Observatory weather on the left and up to four [KMB](https://www.kmb.hk/) bus arrival times on the right.

中文說明見 [README.zh-Hant.md](README.zh-Hant.md)。

## Features

- Live temperature, daily high / low, and day / night weather background
- Up to 4 KMB routes with ETA in minutes
- First-run Wi‑Fi setup portal (`ESP32 Wifi`)
- Browser config page for routes, weather station, and screen brightness
- Optional 3D-printed case: `kmb-eta 2.8_ box.stl`

## What you need

| Item | Notes |
| --- | --- |
| ESP32-2432S028R (2.8" CYD) | ILI9341 panel, microSD slot |
| microSD card | FAT32, 4 GB or larger is fine |
| USB cable | Data cable for flashing |
| Wi‑Fi | 2.4 GHz only (ESP32 has no 5 GHz) |
| PlatformIO | CLI (`pio`) or PlatformIO IDE |

## Project layout

```text
KMB--ETA-/
├── platformio.ini          # CYD board + TFT_eSPI pin map
├── Arduino                 # Original sketch (kept for reference)
├── src/main.cpp            # Firmware built by PlatformIO
├── *.bmp                   # Screen graphics (copy to SD)
├── test_*.vlw              # Smooth fonts (copy to SD)
└── kmb-eta 2.8_ box.stl    # Optional enclosure
```

## Prepare the SD card

The firmware reads all images and fonts from the SD card root. Format the card as **FAT32**, then copy these files to the **root** (not inside a folder):

| File | Purpose |
| --- | --- |
| `kmb.bmp` | Full-screen splash / base layout (320×240) |
| `day.bmp` | Daytime left panel (166×240) |
| `night.bmp` | Night left panel |
| `cloud.bmp` | Cloudy day left panel |
| `night_cloud.bmp` | Cloudy night left panel |
| `test_3.vlw`, `test_8.vlw`, `test_12.vlw`, `test_16.vlw`, `test_26.vlw`, `test_32.vlw`, `test_45.vlw` | Fonts used by the UI |
| Other `test_*.vlw` | Extra sizes included in the repo; safe to copy all |

Insert the card into the CYD **before** powering it. If the SD fails to mount, the screen turns red and the serial log prints `SD Fail`.

## Build and flash

1. Install [PlatformIO Core](https://docs.platformio.org/en/latest/core/installation.html).
2. Connect the CYD over USB.
3. If your serial port is not `/dev/cu.usbserial-2110`, edit `upload_port` / `monitor_port` in `platformio.ini` (on Windows use `COM…`, on Linux often `/dev/ttyUSB0`).
4. From the project root:

```bash
pio run -e cyd -t upload
```

Optional full erase (clears Wi‑Fi and saved routes):

```bash
pio run -e cyd -t erase
pio run -e cyd -t upload
```

Serial monitor:

```bash
pio device monitor -b 115200
```

## First-time Wi‑Fi setup

1. Power the board with the prepared SD card inserted.
2. The screen should go white, then show the KMB splash.
3. On your phone or computer, join the open hotspot **`ESP32 Wifi`**.
4. Open the captive portal (or go to `http://192.168.4.1`).
5. Choose your 2.4 GHz home Wi‑Fi and enter the password.
6. After it connects, the display starts updating weather and bus times.

To forget Wi‑Fi later, open the config page and tap **重設 WiFi** (Reset Wi‑Fi), or erase flash as above.

## Day-to-day use

### Open the config page

With the CYD already on your network:

- Preferred: [http://KMB-ETA.local](http://KMB-ETA.local)
- Or use the board’s IP from your router / serial log

### Configure weather location

1. Open the config page.
2. Under **天氣觀測地區**, pick an HKO station (default: 黃大仙).
3. Save with **儲存並更新**.

### Add bus routes (up to 4)

1. Enter a route number, e.g. `243M`.
2. Choose direction: outbound / inbound.
3. Choose service type if needed (regular `1` or special `2`).
4. Wait for the stop list to load, then pick your stop.
5. Tap **＋ 新增至清單**.
6. Repeat for more routes.
7. Tap **儲存並更新** — the board restarts and shows the new list.

### Brightness

Use the brightness slider on the config page. Values range from about `10` to `255`. The setting is stored in flash.

### Screen layout

| Area | Content |
| --- | --- |
| Left | Time, date, current temp, daily max / min, weather background |
| Right | Up to four bus rows: route, destination, ETA minutes |

ETA `0` / `-` means the bus is due; `!` means no ETA available.

Updates run about every 5 seconds while Wi‑Fi is connected.

## Optional case

Print `kmb-eta 2.8_ box.stl` for a simple desk enclosure sized for the 2.8" CYD.

## Troubleshooting

| Symptom | What to try |
| --- | --- |
| Red screen / `SD Fail` | Reseat the card; confirm FAT32 and files on the root |
| White then KMB splash, but stuck | Join `ESP32 Wifi` and finish Wi‑Fi setup |
| Blank / black screen | Confirm SD files; flash the latest firmware; backlight is GPIO 21, active high |
| No weather / bus data | Check internet access; confirm HKO / KMB APIs reachable |
| Wrong colours or garbled UI | Board must be the ILI9341 2.8" CYD pinout used in `platformio.ini` |
| `KMB-ETA.local` not found | Use the IP address instead; some phones need `.local` / mDNS support |
| Flash port not found | Update `upload_port` in `platformio.ini` |

## Data sources

- Weather: [Hong Kong Observatory Open Data](https://data.weather.gov.hk/)
- Bus ETA: [KMB / LWB ETA API](https://data.etabus.gov.hk/)

This project is unofficial and not affiliated with KMB or HKO.

## License / credit

Firmware based on the original Arduino sketch in this repository. Build configuration targets the community CYD (ESP32-2432S028R) pin map.
