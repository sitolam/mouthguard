# Changelog

All notable changes to MouthGuard are documented here.

## [1.0.0] — 2026-05-17

### Initial release

#### Core detection
- MediaPipe Face Mesh loaded via CDN, running fully client-side
- Lip gap measured using landmarks 13 (upper) and 14 (lower) in pixel space
- Configurable sensitivity threshold (1–20 px, default 5)
- Configurable alert delay (1–10 s, default 1 s)
- Live video feed with mirrored overlay showing lip landmarks and gap line (green = closed, red = open)
- Countdown bar at bottom of video fills as mouth stays open, fires alert when full

#### Background tab support
- Detection loop driven by an inline Web Worker `setInterval` instead of `requestAnimationFrame`, so it keeps running at full speed even when the tab is not focused

#### Alerts
- **Tab title blinking** — primary background alert, works from any origin including `file://`
- **Browser notifications** — fires when served from `localhost` or `https://`, requires permission
- **In-page flash** — red border pulse on the video feed
- **Toast message** — "Close your mouth!" slides in from top-right
- Alert re-arms automatically: after firing, `mouthOpenSince` resets so the countdown refills before the next alert

#### Audio
- Web Audio API synthesized sounds — no external audio files needed
- Five sound types: Soft beep (sine), Chime (3-harmonic), Double beep, Buzz (sawtooth), High ping (descending sine)
- Volume slider (1–100%, default 85%)
- Test button to preview the selected sound immediately

#### Dashboard
- Live stepped timeline chart showing open/closed state over the last 5 minutes (Chart.js)
- Doughnut chart with open/closed percentage breakdown
- Per-session stats: time closed, time open, open event count, average open duration
- Session timer shown in video overlay

#### Session history
- Up to 30 past sessions persisted to `localStorage`
- Each entry shows date/time, session duration, and % open as a colour-coded bar
- Clear history button

#### Settings (persisted to `localStorage`)
- Sensitivity threshold slider
- Alert delay slider
- Sound type selector
- Volume slider
- Audio on/off toggle

#### Deployment
- Single `index.html` file — no build step, no backend
- `docker-compose.yml` with Nginx Alpine for one-command local serving
