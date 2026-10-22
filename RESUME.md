# Resume plan — first hardware test with the new Peak Pro

Paused 2026-10-22. Old unit ("BARRYS GAURDIAN") has a dead BLE radio
(advertised but refused all connections, then went silent after resets).
Everything below the Bluetooth chooser is **untested against real hardware**:
auth handshake, telemetry reads, heat/profile/stealth controls.

## Test checklist (new unit, PC first)

1. Charge the new Peak, tap its button so it's awake.
2. Open https://barrynewman1997-sys.github.io/puffco-remote/ in Chrome on the PC.
3. Click **INITIALIZE UPLINK** (filtered scan). If the device isn't listed,
   use **WIDE SCAN** — a factory-fresh unit should advertise as "Peak Pro"
   or similar.
4. Watch the log (auto-opens on failure; mirrored to DevTools console as
   `[PeakHUD]`). Expected sequence:
   `connecting → GATT connected → locating Lorax service → chars found →
   notifications on → auth OK → initial reads → polling`.
5. First likely failure point is **auth** (GET_SEED 0x00 → SHA-256 token →
   UNLOCK 0x01). If auth fails, capture the exact log lines — could be a
   newer firmware variant of the Lorax protocol.
6. Then verify, in order: live temp/state/battery, heat cycle start/stop,
   boost, profile read/switch/edit, stealth, lantern, brightness.
7. Once PC works: repeat on iPhone via **Bluefy** (the in-app LOG panel is
   the only console there), then do the home-screen Shortcuts setup
   (icon at /puffco-remote/icon.png — still not done).

## Debug tooling already in place

- Every connect step logs to the in-app log AND the browser console
  (`[PeakHUD]` prefix) — readable remotely via claude-in-chrome
  `read_console_messages` with pattern "PeakHUD" on the open tab.
- GATT connect retries ×4 with backoff and a 15s timeout per attempt.
- RELINK button appears if Chrome remembers a previous device grant
  (`navigator.bluetooth.getDevices()`).
- `#demo` hash = full dashboard with simulated telemetry, no device needed.

## Gotchas learned the hard way

- Chrome auto-cancels the Bluetooth chooser for automated clicks — the
  device-pick click must be the user's real click.
- GitHub Pages caches HTML ~10 min; cache-bust with `?fresh=N` when testing
  a fresh deploy.
- The hologram is the literal `peak-render.png` — do NOT replace it with
  generated/traced art.
