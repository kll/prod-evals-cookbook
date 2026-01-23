# Environment Setup

This lesson gets your development environment ready with all the infrastructure Ask Acme needs.

## What We're Setting Up

| Component | Purpose | How |
|-----------|---------|-----|
| **PostgreSQL** | Analytics database (customer data, metrics) | Docker |
| **MongoDB** | Vector search (documents with embeddings) | Docker |
| **Python Environment** | Dependencies, LangChain, LangGraph | uv or Docker |

---

## Option A: Local Setup with uv (Recommended)

### Step 1: Start the Databases

Make sure Docker is running, then:

```bash
# From this directory (setup_environment)
docker compose up -d postgres mongodb
```

This starts:
- **PostgreSQL** on port `5432` (user: `acme`, password: `acme`, database: `acme`)
- **MongoDB** on port `27017` (no auth for local dev)

Verify they're running:

```bash
docker compose ps
```

You should see both containers with status "running".

### Step 2: Set Up Python Environment

```bash
# Go to project root (if not already there)
cd /path/to/prod-evals-cookbook

# Install dependencies with uv (creates .venv automatically)
uv sync

# Activate the virtual environment
source .venv/bin/activate  # Linux/macOS
# or: .venv\Scripts\activate  # Windows

# Or run commands directly without activating
uv run python my_script.py
```

> **Note:** The `pyproject.toml` is in the project root, so always run `uv sync` from there.

### Step 3: Configure Environment Variables

Copy the example environment file:

```bash
cp env.example .env
```

Edit `.env` and add your OpenAI API key:

```bash
OPENAI_API_KEY=sk-your-key-here
```

### Step 4: Verify Everything Works

Run the verification script:

```bash
# From this directory (setup_environment)
uv run python verify_setup.py
```

You should see:
```
✓ PostgreSQL connection successful
✓ MongoDB connection successful
✓ OpenAI API key configured
✓ All dependencies installed

Ready to proceed to seed data setup!
```

---

## Option B: Full Docker Setup (No Local Python Required)

If you prefer not to install uv/Python locally, you can run everything in Docker.

### Step 1: Configure Environment Variables

```bash
# From this directory (setup_environment)
cp env.example .env
```

Edit `.env` and add your OpenAI API key:

```bash
OPENAI_API_KEY=sk-your-key-here
```

### Step 2: Start All Services

```bash
# From this directory (setup_environment)
docker compose up -d
```

This starts:
- **PostgreSQL** on port `5432`
- **MongoDB** on port `27017`
- **App container** with Python, uv, and Jupyter on port `8888`

### Step 3: Access Jupyter Notebooks

Open your browser to: **http://localhost:8888**

The notebooks are mounted from your local filesystem, so changes persist.

### Step 4: Run Commands in the Container

```bash
# Verify setup
docker compose exec app uv run python setup_environment/verify_setup.py

# Seed data
docker compose exec app uv run python setup_seed_data/seed_all.py

# Run any Python script
docker compose exec app uv run python <script.py>

# Open an interactive shell
docker compose exec app bash
```

### Step 5: Run Evaluations

```bash
# Run stage 1 golden set evaluator
docker compose exec app uv run python stage_1_golden_sets/evaluator.py

# Run stage 2 labeled scenarios
docker compose exec app uv run python stage_2_labeled_scenarios/evaluator.py
```

---

## Connecting to the Databases

**PostgreSQL:**
```bash
# Using psql
docker compose exec postgres psql -U acme -d acme

# Or with any Postgres client
# Host: localhost, Port: 5432, User: acme, Password: acme, Database: acme
```

**MongoDB:**
```bash
# Using mongosh
docker compose exec mongodb mongosh

# Or with any MongoDB client
# Connection string: mongodb://localhost:27017
```

---

## Troubleshooting

### Docker Issues

**"Cannot connect to Docker daemon"**
- Make sure Docker Desktop is running

**"Port already in use"**
- Another service is using port 5432, 27017, or 8888
- Either stop that service or modify the ports in `docker-compose.yml`

### uv Issues (Option A only)

**"Command not found: uv"**
- Install uv: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Or with Homebrew: `brew install uv`
- Or on Windows: `powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"`

**"Python version not found"**
- uv can install Python for you: `uv python install 3.11`

### Docker App Container Issues (Option B only)

**"Container keeps restarting"**
- Check logs: `docker compose logs app`
- Ensure `.env` file exists with valid `OPENAI_API_KEY`

**"Changes not reflected in container"**
- The project is mounted as a volume, changes should be immediate
- For dependency changes, rebuild: `docker compose build app`

### OpenAI API Key

**"Invalid API key"**
- Double-check your key at https://platform.openai.com/api-keys
- Make sure there are no extra spaces in your `.env` file

---

## Stopping the Services

When you're done working:

```bash
docker compose down
```

To also remove the data volumes (start fresh):

```bash
docker compose down -v
```

---

## Next Step

Once everything is verified, proceed to load the seed data:

```bash
cd ../setup_seed_data
cat README.md
```

## Troubleshooting

### Docker Issues

**"Cannot connect to Docker daemon"**
- Make sure Docker Desktop is running

**"Port already in use"**
- Another service is using port 5432 or 27017
- Either stop that service or modify the ports in `docker-compose.yml`

### uv Issues

**"Command not found: uv"**
- Install uv: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Or with Homebrew: `brew install uv`

**"Python version not found"**
- uv can install Python for you: `uv python install 3.11`

### OpenAI API Key

**"Invalid API key"**
- Double-check your key at https://platform.openai.com/api-keys
- Make sure there are no extra spaces in your `.env` file

## Stopping the Databases

When you're done working:

```bash
docker compose down
```

To also remove the data volumes (start fresh):

```bash
docker compose down -v
```

## Next Step

Once everything is verified, proceed to load the seed data:

```bash
cd ../setup_seed_data
cat README.md
```
