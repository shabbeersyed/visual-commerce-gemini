# Autonomous Supply Chain with Gemini 3 Flash & AlloyDB AI

Build an **agentic supply chain system** that "sees" physical inventory using Gemini 3 Flash (Code Execution), "remembers" millions of parts using AlloyDB AI (ScaNN), and "transacts" using the A2A Protocol.

🔗 **GitHub Repository:** https://github.com/shabbeersyed/visual-commerce-gemini  
🌐 **Live Deployment:** https://your-cloud-run-url.a.run.app

## What You'll Build

A multi-agent system featuring:
- **Vision Agent**: Uses Gemini 3 Flash (MINIMAL thinking) to count inventory items deterministically via code execution, plus Gemini 2.5 Flash Lite for smart query generation with structured outputs
- **Supplier Agent**: Searches millions of parts using AlloyDB ScaNN vector search with real semantic embeddings (Vertex AI text-embedding-005)
- **Control Tower**: Real-time WebSocket UI with automatic image compression for orchestrating autonomous workflows

## Architecture

![Autonomous Supply Chain Architecture](./assets/architecture-diagram.png)

**Key Components:**
- **Control Tower (port 8080):** WebSocket-based UI with automatic image compression for real-time orchestration
- **Vision Agent (port 8081):** Gemini 3 Flash (MINIMAL thinking) with Code Execution + Gemini 2.5 Flash Lite for query generation (API key)
- **Supplier Agent (port 8082):** AlloyDB ScaNN vector search with real semantic embeddings from Vertex AI (GCP credentials)
- **AlloyDB AI:** Enterprise PostgreSQL with ScaNN index and text-embedding-005 for semantic understanding
- **A2A Protocol:** Dynamic agent discovery via `/.well-known/agent-card.json`

**Hybrid Architecture:** Vision Agent uses Gemini API (simple setup, free tier available), while Supplier Agent uses GCP services (enterprise-grade, compliance-ready). Image optimization and intelligent query generation happen automatically.

## Quick Start

### Prerequisites

- Google Cloud Project with billing enabled
- Cloud Shell or local environment with:
  - `gcloud` CLI configured
  - Python 3.9+
  - Git

### Setup & Run

```bash
# 1. Clone the repository
git clone https://github.com/shabbeersyed/visual-commerce-gemini.git
cd visual-commerce-gemini
# 2. Run setup (validates environment, enables APIs, creates .env)
sh setup.sh

# 3. Provision AlloyDB (if not already done)
# See codelab for easy-alloydb-setup instructions

# 4. Set up database schema and data in AlloyDB Studio
# See codelab for SQL steps

# 5. Start all services
sh run.sh
```

> **📌 Note:** All commands assume you're in the repo root (`visual-commerce-gemini/`). If commands fail with "No such file", verify your location with `pwd` and navigate back to the repo.

Open http://localhost:8080 to see the Control Tower.

## Repository Structure

```
visual-commerce-gemini-3-alloydb/
├── README.md                    # You are here
├── setup.sh                     # Environment setup script
├── run.sh                       # Service startup script
├── cleanup.sh                   # Resource cleanup script
├── .env.example                 # Environment variables template
│
├── agents/                      # Agentic components
│   ├── vision-agent/            # Gemini 3 Flash vision analysis
│   └── supplier-agent/          # AlloyDB ScaNN inventory search
│
├── frontend/                    # FastAPI + WebSocket Control Tower
│   ├── app.py                   # Main server
│   └── static/                  # Real-time UI
│
├── database/                    # AlloyDB schema & seeding
│   ├── seed.py                  # Backup seeding script (uses AlloyDB Connector)
│   └── seed_data.sql            # Schema + data (for AlloyDB Studio)
│
├── test-images/                 # Sample warehouse images for testing
│
└── logs/                        # Runtime logs (gitignored)
    ├── vision-agent.log
    ├── supplier-agent.log
    └── frontend.log
```

## What Each Command Does

### `sh setup.sh`

1. **Validates environment** — Checks gcloud, APIs, project settings, Python 3
2. **Enables APIs** — AlloyDB, Vertex AI, Compute Engine, Service Networking
3. **Configures Gemini** — Prompts for your Gemini API key
4. **Detects AlloyDB** — Auto-discovers instance URI or prompts for input
5. **Creates .env** — Generates configuration file

### `sh run.sh`

1. **Launches Vision Agent** — Port 8081 (Gemini 3 Flash MINIMAL thinking + Gemini 2.5 Flash Lite query generation)
2. **Launches Supplier Agent** — Port 8082 (AlloyDB ScaNN via Python Connector)
3. **Starts Control Tower** — Port 8080 (FastAPI + WebSocket UI with automatic image compression)

## Database Connection

The Supplier Agent connects to AlloyDB via the **AlloyDB Python Connector** (no Auth Proxy needed):

```python
from google.cloud.alloydbconnector import Connector

connector = Connector()
conn = connector.connect(
    inst_uri,         # projects/PROJECT/locations/REGION/clusters/CLUSTER/instances/INSTANCE
    "pg8000",         # Driver
    user="postgres",
    password=DB_PASS,
    ip_type="PUBLIC",  # Use "PRIVATE" for Cloud Run
)
```

This handles IAM authentication, SSL/TLS, and connection routing automatically.

## Key Technologies

- **Gemini 3 Flash** — AI model with MINIMAL thinking level and code execution for deterministic vision analysis
- **Gemini 2.5 Flash Lite** — Fast LLM for semantic query generation with structured outputs (Pydantic models)
- **AlloyDB AI** — PostgreSQL-compatible database with ScaNN vector search (10x faster than HNSW)
- **AlloyDB Python Connector** — Secure connection without Auth Proxy (IAM auth, managed SSL)
- **Vertex AI text-embedding-005** — Real semantic embeddings for accurate similarity matching
- **A2A Protocol** — Agent-to-Agent communication standard for plug-and-play agent composition
- **FastAPI** — Modern Python web framework with WebSocket support and PIL-based image compression

## Troubleshooting

### Port conflicts

```bash
lsof -ti:8080 | xargs kill -9
lsof -ti:8081 | xargs kill -9
lsof -ti:8082 | xargs kill -9
```

### AlloyDB connection issues

**Symptom**: `Connection refused` or `AlloyDB not configured`

**Common causes:**
1. **AlloyDB not configured** — Check `.env` has correct `ALLOYDB_REGION`, `ALLOYDB_CLUSTER`, and `ALLOYDB_INSTANCE`
2. **Public IP not enabled** — Enable it in AlloyDB Console → Instance → Edit → Connectivity
3. **Wrong password** — Check `.env`: `cat .env | grep DB_PASS`
4. **Instance not ready** — Wait 1-2 minutes after provisioning

### Agent not responding

```bash
curl http://localhost:8081/health
curl http://localhost:8082/health
curl http://localhost:8080/api/health
```

## 🎁 Bonus: Deploy to Cloud Run

> **Optional** — Everything works locally, but if you want to share your creation with a public URL:

```bash
sh deploy/deploy.sh
```

The script reads your `.env`, asks for your name, and deploys to Cloud Run. When anyone opens your URL, they'll see a popup:

> 🚀 **Deployed by *Shabbeer Basha Syed*** — Powered by Gemini 3 Flash · AlloyDB AI · A2A Protocol
> *Completed by Shabbeer Basha Syed as part of Code Vipassana Season 14*
> **[Try the codelab yourself →]**

After dismissing, a persistent bottom badge stays: *"Deployed by Your Name · Code Vipassana S14 · Learn how →"*

> **Completed as part of [Code Vipassana Season 14](https://www.codevipassana.dev/)**

## Cleanup

To avoid charges, run the cleanup script:

```bash
sh deploy/cleanup.sh
```

This deletes the AlloyDB cluster, removes any deployed Cloud Run services, and optionally removes local files (logs, `.env`).

If you prefer a manual approach:

```bash
gcloud alloydb clusters delete YOUR_CLUSTER_NAME \
  --region=YOUR_REGION \
  --force
```

## Technical References

### **Official Documentation & Performance Benchmarks**

**Gemini 3 Flash:**
- Code Execution API: https://cloud.google.com/vertex-ai/generative-ai/docs/model-reference/code-execution-api
- Developer Guide: https://ai.google.dev/gemini-api/docs/gemini-3
- Model Documentation: https://docs.cloud.google.com/vertex-ai/generative-ai/docs/models/gemini/3-flash
- Pricing: https://ai.google.dev/gemini-api/docs/pricing

**AlloyDB ScaNN Performance (All claims verified from official sources):**
- ScaNN vs HNSW Benchmarks: https://cloud.google.com/blog/products/databases/how-scann-for-alloydb-vector-search-compares-to-pgvector-hnsw
  - ✅ 10x faster filtered search (when indices exceed memory)
  - ✅ 4x faster standard search
  - ✅ 3-4x smaller memory footprint
  - ✅ 8x faster index builds
- Understanding ScaNN: https://cloud.google.com/blog/products/databases/understanding-the-scann-index-in-alloydb
- AlloyDB Python Connector: https://github.com/GoogleCloudPlatform/alloydb-python-connector
- AlloyDB AI Documentation: https://cloud.google.com/alloydb/docs/ai
- Best Practices: https://docs.cloud.google.com/alloydb/docs/ai/best-practices-tuning-scann

**A2A Protocol:**
- Agent cards at `/.well-known/agent-card.json` (emerging standard)
- Standardized agent discovery and communication

**Additional Context:**
- ScaNN is based on 12 years of Google Research and powers Google Search and YouTube at billion-scale
- Released for general availability: October 2024
- First PostgreSQL vector index suitable for million-to-billion vectors

## 👤 Author

Shabbeer Basha Syed  
Master’s in Information Systems & Technology  
University of North Texas  
