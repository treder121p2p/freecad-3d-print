# FreeCAD 3D Print — AI-Powered 3D Modeling in Docker

Complete, self-contained stack: FreeCAD 1.1.3 + AI assistant (MiMo/Xiaomi via Polza) + Web UI.

**One command to deploy, one browser tab to model.**

## Quick Start

```bash
# 1. Clone
git clone https://github.com/treder121p2p/freecad-3d-print.git
cd freecad-3d-print

# 2. Configure (Polza API key required)
cp .env.example .env
# Edit .env — set POLZA_API_KEY

# 3. Run
docker compose up -d

# 4. Open
# http://localhost:9876
```

That's it. You'll see FreeCAD's 3D view on the left and a chat panel on the right. Type natural language commands in Russian or English.

## What's Inside

```
┌─────────────────────────────────────────────┐
│  Docker Container                           │
│                                             │
│  ┌──────────────┐  ┌─────────────────────┐  │
│  │ FreeCAD 1.1.3 │  │ noVNC (VNC viewer) │  │
│  │ GUI + RPC 9875│←─│ WebSocket proxy    │  │
│  └──────┬───────┘  └─────────────────────┘  │
│         │                                   │
│  ┌──────┴───────┐  ┌─────────────────────┐  │
│  │ AI Bridge    │←─│ MiMo (Polza API)    │  │
│  │ Python 9877  │  │ NL → FreeCAD code   │  │
│  └──────────────┘  └─────────────────────┘  │
│                                             │
│  ┌──────────────────────────────────────┐   │
│  │ Web UI (Node.js 9876)                │   │
│  │ 2/3 Screen + 1/3 Chat                │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

## Ports

| Port | Service | Access |
|------|---------|--------|
| **9876** | **Web UI** (main) | `http://localhost:9876` |
| 9875 | FreeCAD RPC | XML-RPC API |
| 6080 | noVNC (direct) | `http://localhost:6080/vnc.html` |

## Chat Commands (examples)

### Russian
- "Создай куб 20x30x40 мм"
- "Просверли отверстие r=10 по центру пластины"
- "Покажи список объектов"
- "Экспортируй в STL"
- "Удали Cylinder"

### English
- "Create a box 20x30x40 mm"
- "Drill a hole r=10 through the center"
- "List all objects"
- "Export to STL"

The AI (MiMo) understands natural language, asks clarifying questions when needed (dimensions, position), and generates proper FreeCAD Python code.

## Configuration

Edit `.env`:

```bash
# Required: Polza API key
# Get yours at https://polza.ai
POLZA_API_KEY=pza_your_key_here

# Optional: AI model (default: xiaomi/mimo-v2.5)
POLZA_MODEL=xiaomi/mimo-v2.5

# Optional: ports
WEBUI_PORT=9876
RPC_PORT=9875
NOVNC_PORT=6080
```

## RPC API (for automation)

```python
import xmlrpc.client
proxy = xmlrpc.client.ServerProxy('http://localhost:9875')

# Execute FreeCAD Python code
result = proxy.execute_code('''
import FreeCAD, Part
doc = FreeCAD.activeDocument() or FreeCAD.newDocument("MyDoc")
box = doc.addObject("Part::Box", "MyBox")
box.Length = 50; box.Width = 30; box.Height = 20
doc.recompute()
''')

# Create structured objects
proxy.create_object('MyDoc', {
    'Type': 'Part::Cylinder',
    'Name': 'MyCylinder',
    'Properties': {'Radius': 10.0, 'Height': 25.0}
})

# List objects
objects = proxy.get_objects('MyDoc')
```

## Production Deployment

```bash
# Build and run
docker compose up -d --build

# View logs
docker compose logs -f

# Restart
docker compose restart

# Stop
docker compose down

# Update
git pull && docker compose up -d --build
```

### Firewall

Only port 9876 needs to be open for web access. Ports 9875 (RPC) and 6080 (noVNC) can be restricted to local access.

### System Requirements

- Docker + Docker Compose v2
- 4 GB RAM minimum (FreeCAD is memory-heavy)
- 2 CPU cores
- 2 GB disk (Docker image ~1.5 GB)

## License

MIT
