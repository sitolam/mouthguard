# MouthGuard

A browser-based mouth closure tracker that uses your webcam and AI face detection to help you keep your mouth closed. Runs entirely client-side — no backend, no data leaves your device.

![MouthGuard screenshot](https://placehold.co/900x500/0d0f14/818cf8?text=MouthGuard)

## Features

- **Real-time detection** — MediaPipe Face Mesh runs on every webcam frame, measuring the gap between lip landmarks 13 and 14
- **Smart alerts** — configurable detection window: the mouth must stay open for the full delay before anything is registered, so brief openings never trigger false alerts
- **Tab title blinking** — flashes the browser tab title when your mouth has been open too long, visible even when you've switched to another tab
- **Browser notifications** — system notifications when served over `http://localhost` or `https://`
- **Audio alerts** — five synthesized sounds (Soft beep, Chime, Double beep, Buzz, High ping) with adjustable volume
- **Live lip gap chart** — scrolling real-time plot of the lip gap in pixels with a threshold reference line
- **Session stats** — time open/closed, number of events, average open duration
- **Session history** — last 30 sessions persisted to `localStorage`
- **Dark theme** — clean, focused UI with Inter + JetBrains Mono fonts

## Quick start

### Docker image (recommended)

Pull and run the pre-built image:

```bash
docker run -p 8080:80 ghcr.io/sitolam/mouthguard:latest
```

Pin to a specific version:

```bash
docker run -p 8080:80 ghcr.io/sitolam/mouthguard:v1.1.0
```

Or with Docker Compose — create a `docker-compose.yml`:

```yaml
services:
  mouthguard:
    image: ghcr.io/sitolam/mouthguard:latest
    ports:
      - "8080:80"
    restart: unless-stopped
```

```bash
docker compose up -d
```

To update later:

```bash
docker compose pull && docker compose up -d
```

Open [http://localhost:8080](http://localhost:8080).

### Python

```bash
python3 -m http.server 8080
```

### NixOS

```bash
nix-shell -p python3 --run "python3 -m http.server 8080"
```

> **Note:** Opening `index.html` directly as a `file://` URL works for most features, but browser notifications require an `http://` or `https://` origin. Tab title blinking works regardless.

## Settings

| Setting | Default | Description |
|---|---|---|
| Sensitivity threshold | 5 | Lip gap (px) that counts as "open". Lower = more sensitive. |
| Alert delay | 1s | How long mouth must stay continuously open before it is detected and an alert fires. If the mouth closes before the delay expires, nothing is recorded. Range: 0 – 10 s in 0.1 s steps. |
| Sound type | Soft beep | Alert sound character |
| Volume | 85% | Alert sound loudness |

## Tech stack

- Vanilla HTML, CSS, JavaScript — single `index.html` file, no build step
- [MediaPipe Face Mesh](https://chuoling.github.io/mediapipe/solutions/face_mesh.html) via CDN — client-side ML inference
- [Chart.js](https://www.chartjs.org/) via CDN — dashboard charts
- [Nginx](https://nginx.org/) (Alpine) via Docker for serving

## How it works

1. `getUserMedia` captures the webcam stream
2. An inline Web Worker runs a `setInterval` tick loop — workers are not throttled in background tabs, keeping detection alive when you switch away
3. Each tick sends the current video frame to MediaPipe Face Mesh
4. The vertical distance between landmarks 13 (upper lip inner) and 14 (lower lip inner) is measured in pixel space
5. If the gap exceeds the threshold continuously for the full alert delay, the mouth is confirmed as open, an alert fires, and the detection window time is counted as open time. If the gap drops below the threshold before the delay expires, the event is discarded. Once confirmed open, the alert re-arms every alert delay until the mouth closes.
6. On close, the continuous sound and tab blink stop immediately

## Browser support

Tested in Chrome and Firefox. Requires:
- `getUserMedia` (camera access)
- Web Workers
- Web Audio API
- `Notification` API (optional, for system notifications)
