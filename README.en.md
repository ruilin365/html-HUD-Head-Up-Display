# HUD — Head-Up Display

> Turn your phone into a fighter-jet head-up display. Raise the phone and the horizon, heading, speed and altitude follow your motion in real time.

[中文](README.md) | **English**

<img src="./screenshot.png" width="300" alt="HUD screenshot">

## Live demo

<https://hud-headup-display.app.workbuddy.host/?sharecode=L346LRQHPP>

Open it in a mobile browser — nothing to install. Chrome or Safari recommended. Add it to your home screen and it launches without an address bar, close to a native app.

## Features

- **Attitude indicator** — the pitch ladder slides as you tilt up/down, rotates as you bank, with a tick and label every 10°
- **Heading** — three-digit readout on top plus a scrolling compass tape at the bottom
- **Scrolling speed / altitude tapes** — central readout box, digits face outward, imperial (KT · FT) and metric (KPH · M)
- **Flight path vector** — a ring with side wings showing the actual direction of travel; it swings noticeably to one side during turns
- **CRT feel** — phosphor glow layered with scanlines and vignette, two palettes (green / amber)

## Controls

| Gesture | Action |
| --- | --- |
| Tap once | A settings button appears top-right, fades after 3 s |
| Double tap | Toggle metric / imperial |
| Hold 0.9 s | Re-zero the current attitude as the new reference |

The settings panel switches data source, screen orientation, speed multiplier, forward speed, theme and units.

## Settings

| Item | Options | Notes |
| --- | --- | --- |
| Data source | GPS / Accelerometer | GPS uses satellite positioning; accelerometer integrates the motion sensor |
| Orientation | Portrait / Landscape | Landscape is a software rotation, no orientation-lock API needed |
| Speed multiplier | 1× / 10× / 100× | Amplifies the tiny speeds a phone sensor picks up; accelerometer mode only |
| Forward speed | 0 / 10 / 100 m/s | Adds a constant forward vector to simulate sustained flight |
| Theme | Green / Amber | Applies immediately |
| Units | Imperial / Metric | Same as the double-tap gesture |

## Running locally

The whole app is a single `index.html` with no external dependencies and no build step.

**Important: it must be served over `https://` or `localhost`.** Browsers only expose motion sensors and geolocation in a secure context — opening the file with `file://` yields no data and the page falls back to demo mode automatically.

Quickest way to serve it:

```bash
# either one
python3 -m http.server 8080
npx serve .
```

Then open <http://localhost:8080>. Use `localhost`, not your LAN IP — the latter is not a secure context either.

To use it from another device you need https. No certificate ships with this repo; sign a self-signed one with `openssl` and tap "Proceed" when the browser warns you.

## How it works

- **Attitude** is derived from a full rotation matrix rather than reading `beta` / `gamma` directly. When the phone is held upright, banking around the screen normal changes *both* Euler angles at once, so naive reads break in landscape and when inverted; the matrix formulation holds at any orientation, with `screen.orientation.angle` used to mix components in landscape.
- **Accelerometer mode** integrates gravity-removed linear acceleration into velocity, with damping and bias estimation to curb drift. Physically, acceleration only reflects *changes* in velocity — at constant speed the integral is zero — so the reading decays back toward zero by design.
- **Turn slip** is generated straight from heading angular velocity, so the vector swings out the instant you turn and recentres as soon as you stop.
- **GPS** data is median-filtered with an accuracy gate; altitude is rejected when precision is poor, which keeps the readout from jumping.
- Two stacked canvases: one blurred for glow, one crisp for text.

## Requirements

- **HTTPS or localhost** (prerequisite for sensors and geolocation)
- iOS 13+ asks for "Motion & Orientation" permission on first entry — tap to grant
- Works out of the box on Android Chrome / Edge

## Known limitations

- Accelerometer-mode speed is an integral, so it drifts back to zero during long steady motion
- GPS is usually unavailable indoors; the display falls back to demo data and the status line shows `SIM`
- Accuracy depends entirely on the phone's sensor quality and varies noticeably between devices

## License

[Apache License 2.0](LICENSE)
