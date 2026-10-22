# Puffco Remote (Peak Remote)

Single-file mobile web app (`index.html`) that controls a Puffco Peak Pro over
Bluetooth LE using the Web Bluetooth API. No backend, no build step, no dependencies.

## How it's used (iPhone)

1. Host `index.html` anywhere with HTTPS (GitHub Pages, etc.).
2. On iPhone, open the URL in **Bluefy** (free App Store browser — Safari/Chrome
   on iOS block Web Bluetooth).
3. Wake the Peak Pro (tap its button), hit **Connect to Peak Pro**, pick the
   device in the chooser.

Android: works directly in Chrome; can be added to the home screen as an app.

## Features

- Start / Stop / Boost heat cycles
- Live bowl temperature + operating state + session countdown bar
- Battery %, charging indicator, lifetime dab counter
- 4 heat profiles: view, switch, edit temperature (350–620°F) and duration (15–180s)
- Stealth mode, lantern mode, LED brightness
- °F/°C toggle (persisted), in-app debug log (Bluefy has no devtools)
- Stark-HUD UI: holographic "Unit Schematic" panel showing the exact device
  render (`peak-render.png`) with an animated scan line and live callout chips
  (state, temp, battery), radial thermal gauge, live temp sparkline, stat
  tiles. `#demo` URL hash previews the dashboard with simulated telemetry
  (no device needed).

## Protocol

Puffco "Lorax" BLE protocol (firmware X and later), ported to JS from public
reverse-engineering work:

- https://github.com/Fr0st3h/Puffco-Reverse-Engineering-Writeup
- https://github.com/SoyMilkIsOk/puffco-py

Summary: single GATT service `e276967f-...`, commands written to one
characteristic, replies via notifications on another. Packets are
`seq(u16 LE) + opcode(u8) + payload`. Auth = SHA-256 challenge/response:
token = `SHA256(masterKey + seed[:16])[:16]`. Device values live at virtual
file paths (e.g. `/p/app/htr/temp`, `/p/app/mc`) accessed with READ_SHORT
(0x10) / WRITE_SHORT (0x11). Telemetry is polled (~1.1s fast loop, battery/
dabs every ~8 cycles).

Reference clones (not committed) were reviewed in the Claude scratchpad.

## Status

- 2026-10-20: built, JS syntax-validated. Not yet tested against hardware —
  needs real Peak Pro + Bluefy session. Debug log panel captures all protocol
  traffic for troubleshooting.
- 2026-10-22: PAUSED pending replacement hardware. The original Peak
  ("BARRYS GAURDIAN") has a dead/wedged BLE radio: it appeared in scans but
  rejected every GATT connection (Chrome ×4 retries, Windows-level pairing),
  then stopped advertising entirely after button resets — invisible to both PC
  and iPhone. Not an app bug; connect flow never got past the OS link layer.
  Added this session: step-by-step connect logging (mirrored to console as
  `[PeakHUD]`), GATT retry ×4 with 15s timeout, wide-scan button, one-tap
  relink via remembered grants. See RESUME.md for the test plan when the new
  unit arrives.
