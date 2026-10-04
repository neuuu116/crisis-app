# Crisis Support Hub

A fully offline-capable Progressive Web App (PWA) for crisis support, coping strategies, and anonymous peer support. Built with FastAPI and vanilla JavaScript—no database, no external dependencies, ready to deploy.

## Features

- 🔌 **Offline-First**: Service worker caches all content and API responses. Works completely offline after first visit.
- 🧘 **Coping Strategies**: Guided exercises (4-7-8 breathing, 5-4-3-2-1 grounding, cold water reset) cached locally.
- 📞 **Crisis Resources**: Hotline directory (988, Crisis Text Line, SAMHSA, Emergency Services) with geolocation-based distance sorting.
- 💬 **Anonymous Peer Support**: Post anonymous messages; stays in memory (vanishes on server restart).
- 📱 **Progressive Web App**: Install as a native-like app on mobile or desktop. Works without network.
- ♿ **Accessible**: ARIA labels, semantic HTML, keyboard navigation (Ctrl/Cmd+Enter to post).
- 🚀 **Zero Configuration**: In-memory seed data; no database setup needed.

## Tech Stack

- **Backend**: Python 3.12 + FastAPI
- **Frontend**: Vanilla JavaScript (no framework, no build step)
- **Containerization**: Docker
- **PWA**: Service Worker, Web Manifest, Cache API

## Quick Start

### Local Development

**1. Install dependencies:**
```bash
pip install -r requirements.txt
```

**2. Run the server:**
```bash
python3 main.py
```

Or with auto-reload:
```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

**3. Open your browser:**
```
http://localhost:8000
```

## Project Structure

```
crisis-app/
├── main.py                 # FastAPI app, all API endpoints
├── requirements.txt        # Python dependencies
├── Dockerfile             # Docker build config
├── .dockerignore          # Build exclusions
├── static/
│   ├── index.html        # PWA shell (tabs, layout)
│   ├── app.js            # Vanilla JS app logic, offline outbox
│   ├── sw.js             # Service Worker (caching)
│   ├── styles.css        # Responsive styling
│   ├── manifest.webmanifest  # PWA metadata
│   └── icon.svg          # App icon
└── README.md
```

## API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/health` | GET | Liveness check, post count |
| `/api/bootstrap` | GET | All strategies + resources (cached on first visit) |
| `/api/strategies` | GET | Coping techniques |
| `/api/resources` | GET | Hotlines + in-person services (optional: `?lat=X&lon=Y` for distance sorting) |
| `/api/posts` | GET | Recent anonymous posts (default: 50, max 200) |
| `/api/posts` | POST | Create new post (idempotent via `client_id`) |

### Example Requests

**Fetch strategies:**
```bash
curl http://localhost:8000/api/strategies
```

**Sort resources by distance:**
```bash
curl "http://localhost:8000/api/resources?lat=40.7306&lon=-73.9866"
```

**Post anonymously:**
```bash
curl -X POST http://localhost:8000/api/posts \
  -H "Content-Type: application/json" \
  -d '{"content": "I am struggling", "client_id": "unique-id"}'
```

## Docker Deployment

### Build & Run Locally

```bash
docker build -t crisis-hub .
docker run -p 8000:8000 crisis-hub
```

### Deploy to Cloud

#### Render (Easiest)
1. Push to GitHub
2. Go to [render.com](https://render.com)
3. Click **New → Web Service**
4. Select your GitHub repo
5. Set:
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `uvicorn main:app --host 0.0.0.0 --port $PORT`
6. Click **Deploy**

#### Railway
```bash
railway link
railway up
```

#### Fly.io
```bash
flyctl launch
flyctl deploy
```

#### Docker Hub + Manual VPS
```bash
docker login
docker build -t your-username/crisis-hub:latest .
docker push your-username/crisis-hub:latest

# On your VPS:
docker run -d -p 8000:8000 your-username/crisis-hub:latest
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8000` | Server port (Render/Railway/Fly auto-set this) |

## Offline Behavior

1. **First Visit**: App downloads strategies, resources, and service worker. All cached.
2. **Offline Mode**: 
   - All cached content loads instantly.
   - Posts queue locally in `localStorage` (outbox pattern).
   - Network status indicator shows "Offline" in header.
3. **Back Online**: 
   - Queued posts auto-retry.
   - Fresh post list fetched.
   - Geolocation sorting works again.

## Development

### Adding New Coping Strategies

Edit `STRATEGIES` in `main.py`:
```python
{
    "id": 4,
    "title": "Progressive Muscle Relaxation",
    "duration": "~5 min",
    "steps": [
        "Tense your toes for 5 seconds, then relax.",
        "Move up through each muscle group...",
    ],
}
```

### Adding New Resources

Edit `RESOURCES` in `main.py`:
```python
{
    "id": 6,
    "name": "Your Local Clinic",
    "kind": "in_person",
    "phone": "555-0123",
    "lat": 40.7128,
    "lon": -74.0060,
    "note": "Open 9–5, weekdays",
}
```

### Customizing the Frontend

- **Styles**: Edit `static/styles.css`
- **Layout**: Edit `static/index.html`
- **Logic**: Edit `static/app.js`
- **Icons**: Replace `static/icon.svg`

No build step needed—changes take effect on page refresh (or hard refresh).

## Important Notes

⚠️ **Student Demo**: This is a hackathon/educational project, not a monitored crisis service. All posts live in server memory and vanish on restart.

⚠️ **Emergency**: If you are in crisis, call your local emergency number or:
- **US**: Call/text **988** (24/7)
- **UK**: Call **116 123**
- **Global**: Visit [findahelpline.com](https://findahelpline.com)

## License

MIT

## Author

Neha Mhatre  
[GitHub](https://github.com/neuuu116/crisis-app) | [Portfolio](https://neha-portfolio.com)
