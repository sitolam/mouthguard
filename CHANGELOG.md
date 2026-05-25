# Changelog

All notable changes to MouthGuard are documented here.

## [1.3.0] — 2026-05-25

### New features

#### Distance compensation
- **Distance compensation toggle** — when enabled, the raw lip gap is normalised by face height (forehead landmark 10 → chin landmark 152) and scaled to a reference face size before being compared to the threshold. This keeps sensitivity consistent regardless of how far you sit from the camera; toggling it off restores the previous pixel-gap behaviour. Persisted to `localStorage`.
- **Calibrate button** — sit at your preferred distance, start a session, and click Calibrate to capture the current face height as the reference. The reference is shown in the setting row and persisted across page reloads. Button briefly confirms with `✓ Set (NNNpx)` on success, or `No face detected` if no face is visible.
- **Dashboard distance readout** — three small readings appear below the Live Lip Gap chart (hidden when compensation is off): **Face size** (current face height in px), **Ref size** (the saved calibration reference), and **Adj. gap** (the compensated gap value being compared to the threshold). The Live Lip Gap chart plots the adjusted gap when compensation is active so the threshold line stays meaningful.

## [1.2.2] — 2026-05-19

### Improvements

- **Alert delay is now the detection window** — the mouth must stay continuously open for the full alert delay before the system registers it as open at all. Previously a 100 ms debounce confirmed state and then the alert delay was a separate countdown on top. Now there is a single configurable window: if the mouth closes before the delay expires, nothing is recorded and no alert fires, eliminating false positives from brief openings.
- **Detection window counted as open time** — when a mouth-open event is confirmed, the time spent in the detection window is retroactively added to the "time open" total so stats accurately reflect the full duration the mouth was open.
- **Alert delay description updated** — the setting now reads "Mouth must stay open this long to be detected & trigger an alert" to match the new behaviour.

## [1.2.1] — 2026-05-17

### Bug fixes

- **Stop session button** — clicking Stop no longer immediately restarts the session. The root cause was an `addEventListener('click', startSession)` added at init that was never removed; it fired alongside the `onclick = stopSession` assignment, causing both handlers to run simultaneously. Fixed by using `onclick` exclusively.
- **False open/closed readings** — added a 100 ms debounce before confirming a mouth state transition. The camera occasionally produces a single-frame blip that would flip state and count a bogus open event. The confirmed state now only changes if the raw signal holds steady for 100 ms; time accounting (open/closed totals) uses the confirmed state throughout.
- **Session history clarity** — the percentage column now reads `X% open` instead of a bare `X%`, and a subtitle `bar & % = time mouth was open` has been added under the panel title, making the metric immediately clear without needing to guess.

## [1.2.0] — 2026-05-17

### Improvements

#### Alert delay
- Minimum alert delay lowered from 1 s to **0 s** (fires instantly when mouth opens)
- Step precision increased from 1 s to **0.1 s** — e.g. 0.5 s, 1.5 s, 2.3 s are all valid
- Display value updates to one decimal place when fractional (e.g. `0.5s`, `1.2s`)

#### Live Lip Gap chart (replaces Live Timeline)
- Chart now plots the **actual lip gap in pixels** on every detection frame (~7 fps) instead of a binary open/closed step
- X axis shows elapsed session time in seconds (`0s` → current)
- Y axis shows gap in `px` with auto-scaling
- A dashed red **threshold line** overlays the chart so you can see exactly how close the gap is to your alert threshold at any point
- Rolling 60-second window (was 5 minutes of coarse data)

## [1.1.0] — 2026-05-17

### Deployment

#### Docker
- Added `Dockerfile` using `nginx:alpine` — bakes `index.html` directly into the image (no volume mount required)
- GitHub Actions workflow (`.github/workflows/docker.yml`) builds and pushes to `ghcr.io` on version tags (`v*`)
- Image is tagged with both `:latest` and the exact tag name (e.g. `:v1.1.0`)

#### Using the image

**Docker run** — pull and serve on port 8080:
```bash
docker run -p 8080:80 ghcr.io/sitolam/mouthguard:latest
```
Then open http://localhost:8080 in your browser.

Pin to a specific version:
```bash
docker run -p 8080:80 ghcr.io/sitolam/mouthguard:v1.1.0
```

**Docker Compose** — create a `docker-compose.yml` anywhere on your machine:
```yaml
services:
  mouthguard:
    image: ghcr.io/sitolam/mouthguard:latest
    ports:
      - "8080:80"
    restart: unless-stopped
```
Then run:
```bash
docker compose up -d
```
To update to the latest image later:
```bash
docker compose pull && docker compose up -d
```

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
