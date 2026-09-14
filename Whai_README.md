# revamp-project-summaries
The projects I have PMed, without sharing confidential files

# WHAI Technologies - Agent Orchestration System

A production-ready multi-agent orchestration platform with real-time monitoring dashboard, knowledge base integration, and comprehensive analytics.

## Project Structure

```
sp26-whai-technologies/
├── agents/                    # Agent implementations (Router, Planner, Executor, Verifier)
├── backend/                   # Dashboard backend API (FastAPI)
│   ├── backend.py            # Main API server with task endpoints
│   ├── kb_manager.py         # Knowledge base management router
│   ├── requirements.txt      # Python dependencies
│   ├── start_backend.bat     # Windows startup script (cmd)
│   └── start_backend.ps1     # Windows startup script (PowerShell)
├── channels/                  # Communication adapters (Console, Slack, WhatsApp)
├── core/                      # Core orchestration engine
│   ├── api.py               # Core API interfaces
│   ├── contracts.py         # Data contracts and models
│   ├── conversation.py      # Conversation management
│   ├── engine.py            # Orchestration engine
│   ├── ingestion.py         # Message ingestion
│   ├── models.py            # Data models
│   └── store.py             # Persistent storage
├── docs/                      # Documentation
│   ├── CLIENT_DEMO_GUIDE.md          # 20-min client demo script
│   ├── DEMO_QUICK_REFERENCE.md       # One-page demo cheat sheet
│   ├── IMPLEMENTATION_SUMMARY.md     # Feature implementation audit
│   ├── TESTING_GUIDE.md              # Comprehensive testing instructions
│   └── WEEK3_DEMO.md                 # Week 3 demo notes
├── orchestration/             # Orchestration logic and workflows
├── skills/                    # Agent skills (Analytics, Support, Local tools)
├── Task_Monitor/              # React dashboard frontend
│   ├── demo_documents/       # Sample docs for demo
│   ├── public/               # Static assets
│   ├── src/                  # React source code
│   │   ├── components/       # React components (KnowledgeBase, Analytics)
│   │   ├── Dashboard.jsx     # Main dashboard component
│   │   ├── main.jsx          # React entry point
│   │   └── App.css           # Tailwind CSS imports
│   ├── src-tauri/            # Tauri desktop app config (optional)
│   ├── uploads/              # KB document uploads
│   ├── index.html            # HTML entry point
│   ├── package.json          # Frontend dependencies
│   ├── vite.config.js        # Vite build config
│   └── README.md             # Frontend-specific docs
├── temporal/                  # Temporal workflow integration
├── tests/                     # Test suites
│   ├── frontend/             # Frontend connectivity tests
│   │   └── test_frontend.html
│   ├── test_agents.py        # Agent unit tests
│   ├── test_channels.py      # Channel adapter tests
│   ├── test_dashboard_backend.py  # Dashboard API tests (11 tests)
│   ├── test_ingestion.py     # Ingestion tests
│   ├── test_pipeline.py      # Pipeline integration tests
│   ├── test_planner.py       # Planner tests
│   └── test_router.py        # Router tests
├── .gitignore                 # Git ignore rules
├── console_client.py          # CLI client for testing
├── database.py                # Core database utilities
├── durability_demo.py         # Durability demonstration
├── main.py                    # Core system entry point
├── populate_db.py             # Sample data generator
├── requirements.txt           # Root Python dependencies
├── start_backend.bat          # Quick start backend (from root)
├── start_backend.ps1          # Quick start backend (from root)
├── start_frontend.bat         # Quick start frontend (from root)
├── start_frontend.ps1         # Quick start frontend (from root)
└── verify_e2e.py              # End-to-end verification
```

---

## Quick Start

### 1. Install Dependencies

```bash
# Backend
pip install -r backend/requirements.txt
pip install -r requirements.txt  # Core system

# Frontend
cd Task_Monitor
npm install
cd ..
```

### 2. Populate Sample Data

```bash
python populate_db.py
```

### 3. Start Services

**Terminal 1 - Backend:**
```bash
.\start_backend.bat
```

**Terminal 2 - Frontend:**
```bash
.\start_frontend.bat
```

### 4. Access Dashboard

Open browser to: **http://localhost:1420**

**For detailed setup instructions, see `SETUP.md`**

---

## System Architecture

### Core Orchestration System

The main agent orchestration system consists of:

1. **Router** (`agents/router.py`) - Routes incoming requests to appropriate agents
2. **Planner** (`agents/planner.py`) - Creates execution plans
3. **Executor** (`agents/execution.py`) - Executes planned actions
4. **Verifier** (`agents/verifier.py`) - Validates execution results

### Dashboard Backend (`backend/`)

FastAPI server providing:
- Task monitoring endpoints (`GET /tasks`, `GET /tasks/{id}`)
- Command ingestion (`POST /ingest_message`)
- Knowledge base management (`POST /kb/upload`, `GET /kb/documents`, `GET /kb/search`)
- Document operations (`DELETE`, `POST` reindex, `GET` metadata)

**Database:** SQLite (`agent_orchestrator.db`)
- `tasks` table - Agent task execution logs
- `documents` table - Uploaded knowledge base documents
- `document_chunks` table - Chunked document content for retrieval

### Dashboard Frontend (`Task_Monitor/`)

React + Vite SPA with 3 main views:

1. **Monitor Tab** - Real-time task tracking with filtering, search, and detail drill-down
2. **Knowledge Base Tab** - Document upload, management, and retrieval testing
3. **Analytics Tab** - Task metrics, stage distribution charts, status breakdowns

**Tech Stack:**
- React 19.1.0
- Vite 7.0.4
- Tailwind CSS 4.2.2
- Lucide React (icons)
- Recharts (data visualization)

---

## Features

### Task Monitoring
- Real-time task updates (3-second polling)
- Search and filter by status/stage
- Detailed execution traces (router → planner → executor → verifier)
- Command ingestion via UI

### Knowledge Base
- Multi-format document upload (PDF, TXT, MD, CSV)
- Automatic chunking (500 chars per chunk)
- Keyword-frequency based relevance scoring
- Document management (delete, re-index, view metadata)
- Retrieval testing interface

### Shopify E-Commerce Connector
- Fetch orders, products, and customers via Shopify Admin REST API
- Sanitized JSON contracts for all three resource types
- Pre-computed plain-text insights (revenue totals, stock health, VIP segments)
- Ready-to-inject LLM prompt templates for deeper analysis
- Natural language routing ("check inventory", "show orders", "list customers")
- Requires `SHOPIFY_ACCESS_TOKEN` and `SHOPIFY_SHOP_DOMAIN` in `.env`

### Analytics
- Task statistics (total, avg execution time, success/failure rates)
- Workflow stage distribution (bar chart)
- Status breakdown (pie chart)
- Top 5 most common task types (ranked list)

### Settings
- Upload and store api keys

### System Health
- Connection status indicator
- Auto-reconnect on backend restart
- Graceful error handling

---

## Testing

### Backend Tests (pytest)

```bash
pytest tests/test_dashboard_backend.py -v
```

**11 tests covering:**
- Task ingestion and retrieval
- Knowledge base upload, search, and management
- Document delete, reindex, and metadata endpoints

### Frontend Tests (browser)

1. Start backend: `.\start_backend.bat`
2. Open `tests/frontend/test_frontend.html` in browser
3. Click "Run All Tests"

**6 tests validating:**
- CORS configuration
- API connectivity
- JSON response formats
- File upload from browser

### Core System Tests

```bash
pytest tests/ -v
```

Tests for agents, channels, ingestion, pipeline, planner, and router.

---

## Demo for Clients

See **`docs/CLIENT_DEMO_GUIDE.md`** for a complete 20-minute demo script including:
- Pre-demo setup checklist
- Detailed talking points for each feature
- Q&A responses
- Troubleshooting tips

Quick reference: **`docs/DEMO_QUICK_REFERENCE.md`** (one-page cheat sheet)

---

## Development

### Project Dependencies

**Core System:**
- Python 3.13+
- FastAPI, Uvicorn
- SQLite
- Pydantic

**Dashboard:**
- Node.js 18+
- React 19
- Vite 7
- Tailwind CSS 4

### Adding New Features

**New agent skill:**
1. Create skill in `skills/local/`
2. Register in skill registry
3. Test with `console_client.py`

**New dashboard tab:**
1. Create component in `Task_Monitor/src/components/`
2. Add tab to `Dashboard.jsx`
3. Add icon to sidebar

**New backend endpoint:**
1. Add route to `backend/backend.py` or `backend/kb_manager.py`
2. Add test to `tests/test_dashboard_backend.py`
3. Update API calls in frontend components

---

## Deployment

### Backend
```bash
# Production mode (no reload)
cd backend
uvicorn backend:app --host 0.0.0.0 --port 8001
```

Deploy to AWS, Azure, GCP, or any platform supporting Python/FastAPI.

### Frontend

**Option 1: Static hosting**
```bash
cd Task_Monitor
npm run build
# Deploy dist/ folder to Netlify, Vercel, S3, etc.
```

**Option 2: Desktop app (Tauri)**
```bash
cd Task_Monitor
npm run tauri build
# Distributable in src-tauri/target/release/
```

---

## Troubleshooting

### PowerShell execution policy error
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Or use `.bat` files instead of `.ps1` files.

### Backend won't start
- Verify Python path in startup scripts
- Check port 8001 is not in use: `netstat -ano | findstr :8001`
- Install dependencies: `pip install -r backend/requirements.txt`

### Frontend shows "Disconnected"
- Backend must be running on port 8001
- Check CORS settings in `backend/backend.py`
- Verify no firewall blocking localhost

### UI looks unstyled
```bash
cd Task_Monitor
npm install
```

### Charts not rendering
```bash
cd Task_Monitor
npm install recharts
```

---

## Documentation

### Getting Started
- **`GETTING_STARTED.md`** - Quick start guide (5 minutes) ⭐ START HERE
- **`SETUP.md`** - Complete installation and setup guide
- **`PROJECT_STRUCTURE.md`** - Visual project structure and file organization
- **`RESTRUCTURE_SUMMARY.md`** - What changed in the reorganization
- **`README.md`** - This file (project overview)

### Testing & Demo
- **`docs/TESTING_GUIDE.md`** - Comprehensive testing instructions
- **`docs/CLIENT_DEMO_GUIDE.md`** - 20-minute client demo script
- **`docs/DEMO_QUICK_REFERENCE.md`** - One-page demo cheat sheet
- **`docs/IMPLEMENTATION_SUMMARY.md`** - Feature audit and implementation details

### Component Documentation
- **`backend/README.md`** - Backend API documentation
- **`Task_Monitor/README.md`** - Frontend documentation
- **`tests/README.md`** - Test suite documentation

---

## License

[Add your license here]

## Contributors

WHAI Technologies Team - SP26

