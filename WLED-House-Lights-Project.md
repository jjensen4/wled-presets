# WLED House Lights — Project Handoff

Updated 2026-10-12 from the live controller. A pre-change copy of `presets.json` is in `Presets/presets-backup-2026-10-12.json`. The file now on the device is `Presets/presets.json`.

## Overview

Holiday and everyday presets for house LED lights on a QuinLED Dig-Quad running Aircoookie WLED.

**Goal:** Keep on-device preset storage modest. The primary control path is a small web app that POSTs a state to the local JSON API. Full `presets.json` swaps remain available for seasonal sets.

## Hardware and network

- **Controller:** Dig-Quad, device name `WLED-House`
- **Firmware:** WLED 16.0.1 “Niji” (build 2606300), ESP32, 4 MB flash
- **Address:** `192.168.68.76` (DHCP). Also `http://wled-house`, `http://wled-house.local`, and `http://WLED-House.local`
- **Web UI:** `http://192.168.68.76`
- **API base:** `http://192.168.68.76/json`
- **Saved static address:** `192.168.0.82` with gateway `0.0.0.0`. That address does not answer. Use `192.168.68.76`.
- **LEDs:** 719 RGB (not RGBW), WS2812, color order RGB
- **Power cap:** 24,000 mA. Per-LED estimate 55 mA. UI target FPS 42.
- **Effects / palettes:** 220 effects, 72 palettes
- **Filesystem:** about 57 KB used of 983 KB before this update. `presets.json` is 15,668 bytes.
- **Boot:** power off, brightness 128, preset 3 “Security Lights”
- **Fallback AP name:** `WLED-QUAD1` (password is not returned by the API)

### LED outputs

| Output | GPIO | Start | Length | Doc name |
|--------|------|-------|--------|----------|
| Bus 1 | 1 | 0 | 383 | Bottom roof line |
| Bus 2 | 16 | 383 | 336 | Top roof line |

The usual presets use one segment across both buses: `start: 0`, `stop: 719`.

Preset 28 “Spot Lights” is separate. It is 11 short solid-white windows, not the two roof lines:

`7–10`, `98–101`, `171–176`, `247–250`, `282–285`, `340–343`, `392–395`, `545–548`, `625–628`, `661–665`, `698–701`.

## What changed on 2026-10-12

The live show was left alone. It was still preset 13 “Halloween” (Twinklecat, brightness 255, segment `0–719`) after the file update.

1. Segment stops of **720** and **800** were set to **719** so they match the real LED count. WLED was already clamping those at runtime. Autumn’s three interleaved segments (starts 0, 3, and 4) now end at 719. Spot Lights and preset 10 were left as they were.
2. Preset 29 “Haloween 2” was restored. On 16.0.1, effect **77** is **Copy Segment**. That preset was saved when 77 was **Meteor Smooth**. Copy Segment was reading custom slider 3 as source segment 16, which does not exist, so the roof faded out. It is now effect **76 Meteor** with the **Smooth** checkbox on (`o3: true`). Colors, palette 3 (`Colors 1&2`), speed, and intensity are unchanged. The stored name still has the original spelling.
3. `wled-holiday-presets.html` default IP is `192.168.68.76`. Halloween Classic uses effect **37 Chase 2**.

## On-device presets

| ID | Name | Range | Effect |
|----|------|-------|--------|
| 1 | Snowflakes | 0–719 | Colortwinkles (74) |
| 2 | Autumn | 3 segments, stop 719 | Noise Pal (107) |
| 3 | Security Lights | 0–719, spacing 10 | Solid (0). Boot preset. |
| 4 | White Twinkle | 0–719 | Twinklefox (80) |
| 5 | St pattys day | 0–719 | Noise 2 (71) |
| 6 | Christmas 2 | 0–719 | Noise 3 (72) |
| 7 | White | 0–719 | Solid (0) |
| 8 | Easter | 0–719 | Blends (115) |
| 9 | Valentines 2 | 0–719 | Flow (110) |
| 10 | Turn Off | 0–100 | Chase (28), master brightness 1. Does not turn the house off. |
| 11 | Christmas Twinkle | 0–719 | Twinklefox (80) |
| 12 | Red, White, Blue | 0–719 | Noise 2 (71) |
| 13 | Halloween | 0–719 | Twinklecat (81). Was the live preset during this update. |
| 18 | Pride | 0–719 | Pride 2015 (63) |
| 19 | Christmas 3 | 0–719 | Blends (115) |
| 20 | Flicker Orange | 0–719 | Candle (88) |
| 22 | Valentines - Brittany | 0–719 | Colorwaves (67) |
| 24 | 3 Colors | 0–719 | Noise 3 (72) |
| 28 | Spot Lights | 11 windows above | Solid (0) |
| 29 | Haloween 2 | 0–719 | Meteor (76), Smooth on |
| 30 | Halloween 3 | 0–719 | Fire Flicker (45) |

Preset 0 is empty. IDs 14–17, 21, 23, and 25–27 are unused.

To turn the house off, POST `{"on":false}`. That is what the web app’s Off button does.

## Effect IDs on this firmware

IDs are for **16.0.1**. Check `GET /json/eff` after any firmware update. Two 16.0 changes already bit this install:

| Old meaning | ID now | What to use |
|-------------|--------|-------------|
| Halloween (retired in 0.14; was “running orange and purple”) | 53 is **Image** (needs a GIF on the filesystem) | **37 Chase 2**, with orange and purple already in the color slots |
| Meteor Smooth | 77 is **Copy Segment** | **76 Meteor** with checkbox 3 (Smooth) enabled |

Other IDs used by the web app, confirmed on this controller:

| Preset | ID | Effect |
|--------|----|--------|
| Solids | 0 | Solid |
| Spooky Green Glow | 2 | Breathe |
| Orange Purple Chase | 28 | Chase |
| Halloween Classic | 37 | Chase 2 |
| Lightning Storm | 57 | Lightning |
| Orange Fire | 66 | Fire 2012 |
| Purple Twinkle | 74 | Colortwinkles |
| Witching Hour Twinklefox | 80 | Twinklefox |
| Halloween Eyes | 82 | Halloween Eyes |

## JSON API

Docs: https://kno.wled.ge/interfaces/json-api/

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/json/state` | GET | Current state |
| `/json/state` | POST | Change the live lights |
| `/json/info` | GET | Version, LED count, filesystem, name |
| `/json/eff` | GET | Effect names. Index is the effect id. |
| `/json/fxdata` | GET | Slider and checkbox labels for each effect |
| `/json/pal` | GET | Palette names |
| `/json/cfg` | GET | Hardware, network, boot defaults. Wi-Fi password is not included. |
| `/presets.json` | GET | Download presets |
| `/upload` | POST | Multipart upload. Field name `data`, filename `presets.json`. |
| `/edit` | — | File browser in the web UI |

Apply a solid color:

```bash
curl -X POST http://192.168.68.76/json/state \
  -H "Content-Type: application/json" \
  -d '{"on":true,"bri":200,"seg":[{"id":0,"start":0,"stop":719,"col":[[255,100,0]],"fx":0}]}'
```

Read-only checks:

```bash
curl -s http://192.168.68.76/json/info
curl -s http://192.168.68.76/json/state
curl -s http://192.168.68.76/presets.json
```

`{"psave":<id>}` writes the **current** live state over that preset. Do not use it to edit a slot unless the lights are already showing the look you want to store. Replacing `/presets.json` does not change the live output until a preset is loaded.

## Web app

File: `wled-holiday-presets.html`

- Groups: Everyday (warm white, cool white, soft amber, off) and Halloween (10 presets)
- Every generated preset uses one segment, `start: 0`, `stop: 719`
- Tap a preset to POST its state to `/json/state`
- Test connection reads `/json/info`
- Default IP is `192.168.68.76`. A saved value in `localStorage` (`wledIp`) overrides that.
- Backup builds a JSON file from the presets **in the page**. It does not download the controller’s `presets.json`. Uploading that backup in `/edit` would replace the on-device list.
- Serve it over HTTP. Opening it as `file://` blocks `fetch()` to the controller.

```bash
python -m http.server 8000
```

Then open `http://<this-computer>:8000/wled-holiday-presets.html` from a device on the same LAN.

## Repo files

- `WLED-House-Lights-Project.md` — this handoff
- `wled-holiday-presets.html` — phone-built controller page
- `Presets/presets-backup-2026-10-12.json` — device file before the stop and Meteor fixes
- `Presets/presets.json` — file uploaded to the controller on 2026-10-12
