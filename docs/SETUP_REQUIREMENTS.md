# TandemAI Setup Requirements

## Complete Environment Variables & Configuration Guide

This document lists all required and optional environment variables to run the ATLAS/TandemAI project.

---

## 📋 Quick Start Checklist

### Minimum Required (Development)
- ✅ **ANTHROPIC_API_KEY** - Claude models (primary agent)
- ✅ **TAVILY_API_KEY** - Web search capabilities
- ✅ **POSTGRES_URI** - Database for state persistence
- ✅ **JWT_SECRET_KEY** - Authentication
- ✅ **CORS_ORIGINS** - Frontend connection
- ✅ **Ollama** - Local judge models (free)

### Recommended (Enhanced Features)
- 🟡 **OPENAI_API_KEY** - GPT models
- 🟡 **GOOGLE_API_KEY** - Gemini models
- 🟡 **LANGSMITH_API_KEY** - Observability/tracing
- 🟡 **FIRECRAWL_API_KEY** - Web scraping
- 🟡 **E2B_API_KEY** - Code execution sandbox

---

## 🔑 API Keys (AI Models)

### Required

#### ANTHROPIC_API_KEY
**Purpose**: Primary LLM for main research agent (Claude Haiku 4.5)  
**Get it**: https://console.anthropic.com/  
**Format**: `sk-ant-api03-...`  
**Cost**: ~$0.80 per million tokens (Haiku 4.5)  
**Used in**: `backend/module_2_2_simple.py`

```bash
ANTHROPIC_API_KEY=sk-ant-api03-your-key-here
```

---

### Optional (Alternative Models)

#### OPENAI_API_KEY
**Purpose**: GPT models (alternative to Claude)  
**Get it**: https://platform.openai.com/api-keys  
**Format**: `sk-proj-...`  
**Cost**: ~$0.15/$0.60 per million tokens (GPT-3.5/4)  
**Used in**: Alternative model configurations

```bash
OPENAI_API_KEY=sk-proj-your-key-here
```

#### GOOGLE_API_KEY
**Purpose**: Gemini models (alternative to Claude)  
**Get it**: https://makersuite.google.com/app/apikey  
**Format**: `AIza...`  
**Cost**: Free tier available, then ~$0.35 per million tokens  
**Used in**: Alternative model configurations

```bash
GOOGLE_API_KEY=AIzaYourKeyHere
```

#### GROQ_API_KEY
**Purpose**: Fast inference with Groq (Llama, Mixtral)  
**Get it**: https://console.groq.com/  
**Format**: `gsk_...`  
**Cost**: Free tier available  
**Used in**: High-speed inference alternative

```bash
GROQ_API_KEY=gsk_YourKeyHere
```

---

## 🛠️ Tool APIs

### Required

#### TAVILY_API_KEY
**Purpose**: Web search for research agents  
**Get it**: https://tavily.com/#api  
**Format**: `tvly-...`  
**Cost**: Free tier: 1000 searches/month  
**Used in**: `backend/module_2_2_simple.py` (Researcher agent)

```bash
TAVILY_API_KEY=tvly-YourKeyHere
```

---

### Optional

#### FIRECRAWL_API_KEY
**Purpose**: Advanced web scraping  
**Get it**: https://firecrawl.dev/  
**Format**: `fc-...`  
**Cost**: Pay-as-you-go  
**Used in**: Future subagent capabilities

```bash
FIRECRAWL_API_KEY=fc-YourKeyHere
```

#### E2B_API_KEY
**Purpose**: Code execution sandbox  
**Get it**: https://e2b.dev/  
**Format**: `e2b_...`  
**Cost**: Free tier available  
**Used in**: Code execution subagent

```bash
E2B_API_KEY=e2b_YourKeyHere
```

#### GITHUB_API_KEY
**Purpose**: Repository operations  
**Get it**: https://github.com/settings/tokens  
**Format**: `ghp_...`  
**Cost**: Free  
**Used in**: GitHub integration subagent

```bash
GITHUB_API_KEY=ghp_YourKeyHere
```

---

## 🗄️ Database Configuration

### Required

#### POSTGRES_URI
**Purpose**: State persistence, checkpoints, thread management  
**Default**: `postgresql://localhost:5432/langgraph_checkpoints`  
**Format**: `postgresql://[user]:[password]@[host]:[port]/[database]`  
**Used in**: All agent state management

**Setup PostgreSQL**:
```bash
# macOS
brew install postgresql
brew services start postgresql

# Create database
createdb langgraph_checkpoints

# Or with custom user
createdb -U atlas -W langgraph_checkpoints
```

**Configuration**:
```bash
# Development (default)
POSTGRES_URI=postgresql://localhost:5432/langgraph_checkpoints

# Production (with credentials)
POSTGRES_URI=postgresql://atlas:secure_password@localhost:5432/langgraph_checkpoints

# Or separate variables
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=langgraph_checkpoints
POSTGRES_USER=atlas
POSTGRES_PASSWORD=your_password
```

---

### Optional (Extended Features)

#### Redis
**Purpose**: Caching, pub/sub for distributed systems  
**Setup**: `brew install redis && brew services start redis`

```bash
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_URL=redis://localhost:6379
```

#### ChromaDB
**Purpose**: Vector database for embeddings  
**Setup**: `pip install chromadb`

```bash
CHROMA_HOST=localhost
CHROMA_PORT=8000
CHROMADB_URL=http://localhost:8000
```

#### Neo4j
**Purpose**: Graph database (experimental)  
**Setup**: Download from https://neo4j.com/download/

```bash
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password
```

---

## 🔍 Observability & Monitoring

### LangSmith (Recommended)

#### LANGSMITH_API_KEY
**Purpose**: Trace all agent executions, debug, monitor  
**Get it**: https://smith.langchain.com/ (Settings > API Keys)  
**Format**: `lsv2_pt_...`  
**Cost**: Free tier: 5K traces/month  
**Used in**: All LangGraph agent runs

```bash
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=lsv2_pt_YourKeyHere
LANGSMITH_PROJECT=tandem-ai-research
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
```

### MLflow (Optional - Advanced)

**Purpose**: Experiment tracking, model versioning  
**Setup**: Requires MLflow server + MinIO

```bash
MLFLOW_TRACKING_URI=http://localhost:5002
MLFLOW_BACKEND_STORE_URI=postgresql://mlflow:password@localhost:5432/mlflow
MLFLOW_ARTIFACT_URI=s3://mlflow-artifacts
MLFLOW_S3_ENDPOINT_URL=http://localhost:9000

# MinIO credentials
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
AWS_ACCESS_KEY_ID=minioadmin
AWS_SECRET_ACCESS_KEY=minioadmin
```

---

## 🎛️ Application Settings

### Required

#### JWT_SECRET_KEY
**Purpose**: Sign JWT tokens for authentication  
**Generate**: `openssl rand -hex 32`  
**Security**: Must be 256+ bits, keep secret

```bash
JWT_SECRET_KEY=your-secure-random-256-bit-key-here
```

#### CORS_ORIGINS
**Purpose**: Allow frontend to connect to backend  
**Format**: Comma-separated URLs (NO SPACES)

```bash
# Development
CORS_ORIGINS=http://localhost:3000,http://localhost:3001,http://localhost:3002,http://localhost:3003

# Production
CORS_ORIGINS=https://yourdomain.com,https://app.yourdomain.com
```

---

### Optional

#### Server Configuration
```bash
API_HOST=0.0.0.0
API_PORT=8000
FRONTEND_URL=http://localhost:3000
```

#### Environment Mode
```bash
NODE_ENV=development
PYTHON_ENV=development
ENVIRONMENT=development
DEPLOYMENT_MODE=development
```

#### Logging
```bash
LOG_LEVEL=INFO
```

#### WebSocket
```bash
CHUNK_SIZE_BYTES=102400
USE_REVERSE_PROXY=false
```

---

## 🤖 Evaluation Configuration

### JUDGE_MODEL (Local Ollama)

**Purpose**: LLM-as-a-judge for evaluation framework  
**Setup**: Install Ollama first

```bash
# Install Ollama
# macOS: brew install ollama
# Linux: curl -fsSL https://ollama.com/install.sh | sh

# Pull judge model
ollama pull qwen3-vl:2b

# Start Ollama server
ollama serve
```

**Configuration**:
```bash
JUDGE_MODEL=qwen3-vl:2b
OLLAMA_HOST=http://localhost:11434
```

**Model Options**:
- `qwen3-vl:2b` - Most reliable (100% success), slower (~30s per eval)
- `qwen3-vl:4b` - Faster (~15s per eval), 81% success rate
- `gemini-2.5-flash` - API-based (requires GOOGLE_API_KEY), fastest but costs money

---

## 🎛️ Feature Flags

```bash
ENABLE_MLFLOW_TRACKING=true
ENABLE_COST_TRACKING=true
ENABLE_AGENT_MEMORY=true
ENABLE_WEB_SEARCH=true
```

---

## 🚀 Complete Setup Example

### Step 1: Copy Environment File

```bash
# From project root
cp .env.example .env
```

### Step 2: Minimum Configuration (.env)

```bash
# === REQUIRED ===
ANTHROPIC_API_KEY=sk-ant-api03-your-real-key
TAVILY_API_KEY=tvly-your-real-key
POSTGRES_URI=postgresql://localhost:5432/langgraph_checkpoints
JWT_SECRET_KEY=$(openssl rand -hex 32)
CORS_ORIGINS=http://localhost:3000,http://localhost:3001

# === LOCAL JUDGE (FREE) ===
JUDGE_MODEL=qwen3-vl:2b
OLLAMA_HOST=http://localhost:11434

# === RECOMMENDED ===
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=lsv2_pt_your-real-key
LANGSMITH_PROJECT=tandem-ai-dev

# === OPTIONAL ===
OPENAI_API_KEY=sk-proj-your-key-if-you-have-it
GOOGLE_API_KEY=AIza-your-key-if-you-have-it
```

### Step 3: Install Dependencies

```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # macOS/Linux
# .venv\Scripts\activate  # Windows

# Install Python dependencies
pip install -r backend/requirements.txt

# Install frontend dependencies (if using UI)
cd frontend && npm install
```

### Step 4: Setup Databases

```bash
# PostgreSQL
createdb langgraph_checkpoints

# Ollama
ollama serve &
ollama pull qwen3-vl:2b
```

### Step 5: Start Services

```bash
# Terminal 1: Backend API
source .venv/bin/activate
python backend/backend_main.py

# Terminal 2: Frontend (optional)
cd frontend && npm run dev

# Terminal 3: Ollama (if not running as service)
ollama serve
```

---

## 💰 Cost Breakdown

### Free Tier Options
- **Ollama (local)**: $0 - Run qwen3-vl:2b locally
- **Tavily**: 1000 searches/month free
- **LangSmith**: 5K traces/month free
- **PostgreSQL**: $0 - Self-hosted
- **GitHub API**: $0

### Paid Services (Optional)
- **Anthropic Claude Haiku 4.5**: ~$0.80 per 1M tokens
- **OpenAI GPT-4**: ~$0.60 per 1M tokens (completion)
- **Google Gemini**: ~$0.35 per 1M tokens
- **Firecrawl**: Pay-as-you-go
- **E2B**: Free tier, then pay-as-you-go

### Estimated Monthly Cost (Moderate Usage)
- **Minimum (local only)**: $0
- **Light usage**: $5-10 (Anthropic + Tavily paid tier)
- **Heavy usage**: $50-100 (Multiple APIs + large trace volume)

---

## 🔐 Security Best Practices

1. **Never commit `.env` files** - Already in `.gitignore`
2. **Use strong JWT secrets** - Generate with `openssl rand -hex 32`
3. **Rotate API keys regularly** - Every 90 days recommended
4. **Use environment-specific keys** - Separate dev/staging/prod keys
5. **Restrict CORS origins** - Only allow known frontend URLs
6. **Enable LangSmith tracing carefully** - Contains sensitive data
7. **Use `.env.local` for local overrides** - Not committed to git

---

## 🐛 Troubleshooting

### "No module named 'backend'"
**Solution**: Ensure you're running from project root with virtual environment activated

### "ANTHROPIC_API_KEY not found"
**Solution**: Check `.env` file exists in project root and is loaded correctly

### "Connection refused to PostgreSQL"
**Solution**: Ensure PostgreSQL is running: `brew services start postgresql`

### "Ollama model not found"
**Solution**: Pull model: `ollama pull qwen3-vl:2b`

### "CORS error from frontend"
**Solution**: Add frontend URL to `CORS_ORIGINS` in `.env`

---

## 📚 Related Documentation

- [README.md](../README.md) - Project overview
- [CLAUDE.md](../CLAUDE.md) - AI assistant guidance
- [docs/diagrams/](./diagrams/) - Visual architecture
- [docs/setup/](./setup/) - Detailed setup guides

---

**Last Updated**: November 2025  
**Maintainer**: TandemAI Core Team

