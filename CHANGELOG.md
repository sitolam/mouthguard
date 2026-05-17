# Changelog

All notable changes to MouthGuard are documented here.

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
