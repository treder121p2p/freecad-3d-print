# FreeCAD AI Bridge — AI-powered 3D Modeling in Docker

FreeCAD 1.1.3 + AI assistant (MiMo/Xiaomi via Polza API) + Web UI.  
**One command to deploy, one browser tab to model.**

---

## ⚡ Quick Start

```bash
# 1. Clone
git clone https://github.com/treder121p2p/freecad-3d-print.git
cd freecad-3d-print

# 2. Configure (Polza API key required)
cp .env.example .env
# Edit .env → set POLZA_API_KEY

# 3. Run
docker-compose up -d

# 4. Open
# WebUI: http://localhost:9876
# noVNC: http://localhost:6080/vnc.html (password: FreeCAD2026)
```

---

## 🎯 What's Inside

```
┌─────────────────────────────────────────────┐
│              Docker Container               │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Xvfb    │→ │  x11vnc  │← │ FreeCAD  │  │
│  │          │  │ localhost │  │ :9875    │  │
│  └──────────┘  └──────────┘  └────┬─────┘  │
│                                    │        │
│  ┌──────────┐  ┌──────────────┐   │        │
│  │ noVNC    │← │  WebUI       │←──┤        │
│  │ :6080    │  │  :9876       │   │        │
│  └──────────┘  └──────┬───────┘   │        │
│                        │           │        │
│                 ┌──────▼───────┐   │        │
│                 │  AI Bridge   │───┘        │
│                 │  :9877       │             │
│                 │  → Polza API │             │
│                 └──────────────┘             │
└─────────────────────────────────────────────┘
```

### Core Stack
| Component | Technology |
|-----------|-----------|
| 3D CAD | FreeCAD 1.1.3 (AppImage) |
| AI Model | MiMo v2.5 / MiMo v2.5 Pro (Xiaomi via Polza) |
| Web UI | Node.js 12+ (split view: 3D + chat) |
| VNC | x11vnc + noVNC (browser-based 3D view) |
| Bridge | Python 3 (threaded HTTP server) |
| Container | Ubuntu 22.04, Docker |

---

## 🚀 Features

### AI-Powered Modeling
- Type natural language → MiMo generates FreeCAD Python code
- Supports Russian and English
- Multi-step workflow tracking
- Auto-retry with error feedback

### Model Selection
| Model | Use Case |
|-------|----------|
| `xiaomi/mimo-v2.5` | Fast, cheap — simple shapes |
| `xiaomi/mimo-v2.5-pro` | Strong — complex models |

### Image Upload
- Attach photos → MiMo analyzes and recreates as 3D model
- Vision-capable with Pro model

### Chat Memory
- Conversations persist across page refreshes
- Server-side session storage
- Export chat as .txt

### Security
- VNC password protected (localhost only)
- API key via .env (never in Docker image)
- CORS restriction
- Docker HEALTHCHECK

---

## 📋 Chat Commands (examples)

### Russian
- «Создай куб 20x30x40 мм»
- «Просверли отверстие r=10 через центр»
- «Покажи список объектов»
- «Экспортируй в STL»
- «Сделай вырез цилиндром»

### English
- "Create a box 20x30x40 mm"
- "Drill a hole r=10 through center"
- "List all objects"
- "Export to STL"
- "Make a cylinder cut"

---

## ⚙️ Configuration

Edit `.env`:

```bash
# Required
POLZA_API_KEY=your_key_here

# Optional
POLZA_MODEL=xiaomi/mimo-v2.5    # or xiaomi/mimo-v2.5-pro
WEBUI_PORT=9876
CORS_ORIGIN=*
```

---

## 🔌 API

### Chat
```bash
curl -X POST http://localhost:9877/api/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Create a cube 10x10x10", "history": []}'
```

### RPC (direct FreeCAD)
```bash
python3 -c "
import xmlrpc.client
proxy = xmlrpc.client.ServerProxy('http://localhost:9875')
print(proxy.execute_code('import FreeCAD; print(FreeCAD.Version())'))
"
```

---

## 🏗️ Production Deployment

```bash
# Build
docker-compose build

# Run
docker-compose up -d

# View logs
docker-compose logs -f

# Restart
docker-compose restart

# Stop
docker-compose down

# Update
git pull && docker-compose up -d --build
```

### Firewall
Only port 9876 (WebUI) needs to be open. Ports 9875 (RPC) and 6080 (noVNC) can be restricted to localhost.

---

## 📦 System Requirements

- Docker + Docker Compose v2
- 4 GB RAM minimum (FreeCAD is memory-heavy)
- 2 CPU cores minimum
- 10 GB disk (Docker image ~2.5 GB)

---

## 📄 License

MIT
