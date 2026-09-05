# Realtime Location Tracker

A web app where every browser tab that opens it shares its GPS location with all other open tabs in real-time. Each device appears as a marker on a shared map, and the map updates live as devices move. When a tab closes, its marker disappears for everyone else.

## Tech Stack

| Layer | Technology |
|---|---|
| Server | Node.js (ESM), Express |
| Real-time | Socket.io |
| Templating | EJS |
| Map | Leaflet.js + OpenStreetMap |
| Config | dotenv |

## Features

- Live GPS tracking via the browser Geolocation API (`watchPosition`)
- Broadcasts location updates to all connected clients over WebSocket
- Per-device markers with popup showing platform, browser, screen resolution, and last-updated time
- Map auto-fits bounds to show all active markers; "Center Map" button to re-center manually
- Markers are removed automatically when a user disconnects

## Project Structure

```
├── public/
│   ├── css/
│   │   ├── leaflet.css
│   │   └── style.css
│   ├── images/          # Leaflet marker assets
│   ├── js/
│   │   ├── app.js       # Client-side Socket.io + map logic
│   │   └── leaflet.js
│   └── views/
│       └── index.ejs
├── server.js            # Express + Socket.io server
├── .env
├── package.json
└── package-lock.json
```

## Getting Started

### Prerequisites

- Node.js v18+
- npm

### Installation

```bash
git clone https://github.com/vishalxtyagi/realtime-location-tracker.git
cd realtime-location-tracker
npm install
```

Create a `.env` file in the root:

```env
PORT=3000
```

### Running

```bash
# Production
npm start

# Development (auto-reload)
npm run dev
```

Open `http://localhost:3000` and allow location access when prompted.

## How It Works

When you open the app, three things happen immediately:

1. **Device info is collected** — the browser reads your platform, user agent, screen resolution, timezone, connection type, battery level, and hardware info, then sends it to the server over a WebSocket (`deviceInfo` event).
2. **GPS tracking starts** — the browser calls `navigator.geolocation.watchPosition`, which fires continuously as you move. Each update sends your `{ latitude, longitude }` to the server (`sendLocation` event).
3. **The server fans out to everyone** — the server takes your location + device info, attaches your socket ID, and broadcasts the combined payload to every connected client (`receiveLocation` event). This means all open tabs — including your own — immediately see the updated position.

On the map side, each socket ID gets its own Leaflet marker. Clicking a marker shows a popup with the device's platform, browser string, screen size, and when it was last seen. When multiple markers are present, the map automatically zooms and pans to fit all of them. There's also a "Center Map" button in the top-right to snap back to that view if you've panned away.

When a tab closes, the server detects the socket disconnect and emits a `userDisconnect` event with that socket's ID. Every other client then removes that marker from the map.

### Socket Event Reference

| Event | Direction | Payload |
|---|---|---|
| `deviceInfo` | client → server | `{ userAgent, platform, screenResolution, timezone, batteryLevel, ... }` |
| `sendLocation` | client → server | `{ latitude, longitude }` |
| `receiveLocation` | server → all clients | `{ id, latitude, longitude, deviceInfo }` |
| `userDisconnect` | server → all clients | `socket.id` |

## License

MIT — see [LICENSE](LICENSE).
