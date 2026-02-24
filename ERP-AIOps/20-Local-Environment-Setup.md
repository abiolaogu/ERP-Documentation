# ERP-AIOps Local Environment Setup

> **Document ID:** ERP-AIOPS-LES-020
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [17-README.md](./17-README.md), [19-CONTRIBUTING.md](./19-CONTRIBUTING.md)

---

## Overview

This document provides a comprehensive guide for setting up a local development environment for ERP-AIOps. By the end of this guide, you will have a fully functional development stack running locally, including the Rust API server, Python AI Brain, Go Gateway, frontend application, and all supporting data stores.

---

## Table of Contents

1. [System Requirements](#1-system-requirements)
2. [Install Docker](#2-install-docker)
3. [Install Rust Toolchain](#3-install-rust-toolchain)
4. [Install Python Environment](#4-install-python-environment)
5. [Install Go](#5-install-go)
6. [Install Node.js](#6-install-nodejs)
7. [Clone and Configure the Repository](#7-clone-and-configure-the-repository)
8. [Start Infrastructure Services](#8-start-infrastructure-services)
9. [Build and Run Rust API](#9-build-and-run-rust-api)
10. [Start the AI Brain](#10-start-the-ai-brain)
11. [Run the Go Gateway](#11-run-the-go-gateway)
12. [Start the Frontend](#12-start-the-frontend)
13. [Development Workflow](#13-development-workflow)
14. [IDE Configuration](#14-ide-configuration)
15. [Troubleshooting](#15-troubleshooting)

---

## 1. System Requirements

### Minimum Hardware

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 4 cores | 8+ cores |
| RAM | 8 GB | 16+ GB |
| Disk | 20 GB free | 50+ GB free (SSD) |
| OS | macOS 13+, Ubuntu 22.04+, Windows 11 (WSL2) | macOS 14+ or Ubuntu 24.04 |

### Port Requirements

Ensure the following ports are available on your machine:

| Port | Service | Protocol |
|------|---------|----------|
| 3000 | Go Gateway | HTTP |
| 5173 | Frontend (Vite dev server) | HTTP |
| 5433 | YugabyteDB (YSQL) | PostgreSQL |
| 6379 | DragonflyDB | Redis |
| 8080 | Rust API | HTTP |
| 8081 | AI Brain (FastAPI) | HTTP |
| 9000 | RustFS (S3 API) | HTTP |
| 9001 | RustFS (Console) | HTTP |
| 7000 | YugabyteDB (YB-Master) | HTTP |
| 9042 | YugabyteDB (YCQL) | Cassandra |

Check for port conflicts:

```bash
# macOS / Linux
for port in 3000 5173 5433 6379 8080 8081 9000 9001; do
  lsof -i :$port > /dev/null 2>&1 && echo "Port $port is in use!" || echo "Port $port is available"
done
```

---

## 2. Install Docker

### macOS

```bash
# Install Docker Desktop
brew install --cask docker

# Start Docker Desktop (from Applications)
open -a Docker

# Verify installation
docker --version
docker compose version
```

**Configure Docker Desktop Resources:**
- Open Docker Desktop > Settings > Resources
- Set Memory: 8 GB (minimum), 12 GB (recommended)
- Set CPU: 4 cores (minimum), 6+ cores (recommended)
- Set Disk: 40 GB

### Ubuntu / Debian

```bash
# Remove old versions
sudo apt-get remove docker docker-engine docker.io containerd runc

# Install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release

# Add Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add current user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify
docker --version
docker compose version
```

### Windows (WSL2)

```powershell
# Install WSL2 (if not already installed)
wsl --install

# Install Docker Desktop for Windows
# Download from https://www.docker.com/products/docker-desktop/
# Enable WSL2 backend in Docker Desktop settings

# Verify from WSL2 terminal
docker --version
docker compose version
```

---

## 3. Install Rust Toolchain

### Install Rust via rustup

```bash
# Install rustup (Rust toolchain manager)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y

# Reload shell environment
source "$HOME/.cargo/env"

# Verify installation
rustc --version    # Should be 1.76+
cargo --version    # Should match rustc version

# Set the correct toolchain version
rustup default stable
rustup update stable
```

### Install Required Components

```bash
# Formatting and linting
rustup component add clippy rustfmt rust-src

# Development tools
cargo install cargo-watch          # Hot-reload for Rust
cargo install cargo-nextest        # Faster test runner
cargo install cargo-deny           # License and advisory checking
cargo install cargo-audit          # Security advisory checking
cargo install cargo-llvm-cov       # Code coverage
cargo install sqlx-cli             # Database migration tool
cargo install sccache              # Compilation cache
```

### Configure sccache (Optional but Recommended)

`sccache` significantly reduces compilation times by caching compiled artifacts:

```bash
# Install sccache
cargo install sccache

# Configure Cargo to use sccache
mkdir -p ~/.cargo
cat >> ~/.cargo/config.toml << 'EOF'
[build]
rustc-wrapper = "sccache"
EOF

# Verify sccache is active
sccache --show-stats
```

### Configure Rust Environment Variables

Add to your shell profile (`~/.zshrc`, `~/.bashrc`, or `~/.bash_profile`):

```bash
# Rust environment
export CARGO_HOME="$HOME/.cargo"
export RUSTUP_HOME="$HOME/.rustup"
export PATH="$CARGO_HOME/bin:$PATH"

# Incremental compilation (faster rebuilds)
export CARGO_INCREMENTAL=1

# Better error messages
export RUST_BACKTRACE=1

# Logging configuration for development
export RUST_LOG=info,aiops=debug
```

---

## 4. Install Python Environment

### Install Python 3.12

**macOS:**
```bash
brew install python@3.12

# Verify
python3 --version    # Should be 3.12.x
```

**Ubuntu:**
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.12 python3.12-venv python3.12-dev

# Verify
python3.12 --version
```

### Set Up Virtual Environment

```bash
cd ERP-AIOps/ai-brain

# Create virtual environment
python3.12 -m venv .venv

# Activate virtual environment
source .venv/bin/activate    # macOS/Linux
# .venv\Scripts\activate     # Windows

# Upgrade pip
pip install --upgrade pip setuptools wheel

# Install production dependencies
pip install -r requirements.txt

# Install development dependencies
pip install -r requirements-dev.txt
```

### Key Python Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `fastapi` | 0.109+ | Web framework |
| `uvicorn` | 0.27+ | ASGI server |
| `pydantic` | 2.6+ | Data validation |
| `scikit-learn` | 1.4+ | Classical ML algorithms |
| `torch` | 2.2+ | Deep learning (LSTM, GNN) |
| `onnxruntime` | 1.17+ | ONNX model inference |
| `prophet` | 1.1+ | Time-series forecasting |
| `langchain` | 0.1+ | LLM orchestration |
| `openai` | 1.12+ | OpenAI API client |
| `anthropic` | 0.18+ | Anthropic API client |
| `sentence-transformers` | 2.5+ | Text embeddings |
| `numpy` | 1.26+ | Numerical computing |
| `pandas` | 2.2+ | Data manipulation |
| `structlog` | 24.1+ | Structured logging |
| `grpcio` | 1.62+ | gRPC framework |
| `pytest` | 8.0+ | Testing framework (dev) |
| `ruff` | 0.3+ | Linting and formatting (dev) |
| `mypy` | 1.8+ | Type checking (dev) |

---

## 5. Install Go

### Install Go 1.22

**macOS:**
```bash
brew install go@1.22

# Verify
go version    # Should be 1.22.x
```

**Ubuntu:**
```bash
# Download and install
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# Verify
go version
```

### Install Go Development Tools

```bash
# Install tools
go install golang.org/x/tools/cmd/goimports@latest
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install github.com/air-verse/air@latest  # Hot-reload for Go
```

### Build the Gateway

```bash
cd ERP-AIOps/gateway

# Download dependencies
go mod download

# Build
go build -o gateway ./cmd/server/

# Verify
./gateway --version
```

---

## 6. Install Node.js

### Install Node.js 20 LTS

**macOS:**
```bash
# Using nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.zshrc  # or ~/.bashrc
nvm install 20
nvm use 20
nvm alias default 20

# Verify
node --version    # Should be v20.x.x
npm --version     # Should be 10.x.x
```

**Ubuntu:**
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify
node --version
npm --version
```

### Install Frontend Dependencies

```bash
cd ERP-AIOps/frontend

# Install dependencies
npm install

# Verify build
npm run build
```

---

## 7. Clone and Configure the Repository

```bash
# Clone the repository
git clone https://github.com/your-org/ERP-AIOps.git
cd ERP-AIOps

# Copy environment configuration
cp .env.example .env

# Review the .env file and customize if needed
# Default values work for local development
```

### Environment Variables Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `RUST_API_HOST` | `0.0.0.0` | Rust API bind address |
| `RUST_API_PORT` | `8080` | Rust API port |
| `RUST_LOG` | `info,aiops=debug` | Rust logging level |
| `AI_BRAIN_HOST` | `0.0.0.0` | AI Brain bind address |
| `AI_BRAIN_PORT` | `8081` | AI Brain port |
| `AI_BRAIN_WORKERS` | `4` | Uvicorn worker count |
| `GATEWAY_PORT` | `3000` | Gateway port |
| `DATABASE_URL` | `postgres://aiops:aiops_dev_password@localhost:5433/aiops` | YugabyteDB connection |
| `DRAGONFLY_URL` | `redis://localhost:6379` | DragonflyDB connection |
| `RUSTFS_ENDPOINT` | `http://localhost:9000` | RustFS S3 endpoint |
| `RUSTFS_ACCESS_KEY` | `minioadmin` | RustFS access key |
| `RUSTFS_SECRET_KEY` | `minioadmin` | RustFS secret key |
| `LLM_PROVIDER` | `local` | LLM provider (local/openai/anthropic) |
| `OPENAI_API_KEY` | _(empty)_ | OpenAI API key (optional) |
| `ANTHROPIC_API_KEY` | _(empty)_ | Anthropic API key (optional) |

---

## 8. Start Infrastructure Services

Start the supporting data stores using Docker Compose:

```bash
# Start infrastructure only (database, cache, object storage)
docker compose -f infra/docker-compose.yml up -d yugabytedb dragonflydb rustfs

# Wait for YugabyteDB to be ready (can take 30-60 seconds)
echo "Waiting for YugabyteDB..."
until docker compose -f infra/docker-compose.yml exec yugabytedb ysqlsh -U aiops -d aiops -c "SELECT 1;" > /dev/null 2>&1; do
  sleep 2
done
echo "YugabyteDB is ready!"

# Verify all infrastructure services
docker compose -f infra/docker-compose.yml ps
```

### Verify Infrastructure

```bash
# Test YugabyteDB connection
docker compose -f infra/docker-compose.yml exec yugabytedb ysqlsh -U aiops -d aiops -c "SELECT version();"

# Test DragonflyDB connection
docker compose -f infra/docker-compose.yml exec dragonflydb redis-cli ping
# Expected: PONG

# Test RustFS
curl -s http://localhost:9000/minio/health/live
# Expected: (empty 200 response)

# Access RustFS console
# Open http://localhost:9001 in browser (minioadmin / minioadmin)
```

### Run Database Migrations

```bash
# Option 1: Using sqlx-cli
export DATABASE_URL="postgres://aiops:aiops_dev_password@localhost:5433/aiops"
cd ERP-AIOps
sqlx migrate run --source migrations/

# Option 2: Using the migration binary
cargo run --bin aiops-migrate -- up

# Verify migrations
sqlx migrate info --source migrations/
```

---

## 9. Build and Run Rust API

### Build All Crates

```bash
cd ERP-AIOps

# Check workspace compiles
cargo check --workspace

# Build all crates in debug mode
cargo build --workspace

# Build the API server binary
cargo build --bin aiops-api
```

### Run the Rust API

```bash
# Run directly
cargo run --bin aiops-api

# Run with hot-reload (recommended for development)
cargo watch -x 'run --bin aiops-api' -w crates/

# Run with specific log level
RUST_LOG=debug cargo run --bin aiops-api
```

**Verify the Rust API:**

```bash
# Health check
curl -s http://localhost:8080/healthz | jq .

# Readiness check
curl -s http://localhost:8080/readyz | jq .

# List API routes (if route listing is enabled)
curl -s http://localhost:8080/api/v1/ | jq .
```

---

## 10. Start the AI Brain

### Activate Virtual Environment

```bash
cd ERP-AIOps/ai-brain
source .venv/bin/activate
```

### Run the AI Brain

```bash
# Run with hot-reload (development)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8081

# Run with multiple workers (closer to production)
uvicorn app.main:app --workers 4 --host 0.0.0.0 --port 8081

# Run with specific log level
LOG_LEVEL=debug uvicorn app.main:app --reload --host 0.0.0.0 --port 8081
```

**Verify the AI Brain:**

```bash
# Health check
curl -s http://localhost:8081/health | jq .

# View auto-generated API docs
# Open http://localhost:8081/docs in browser (Swagger UI)
# Open http://localhost:8081/redoc in browser (ReDoc)

# Test anomaly analysis endpoint
curl -X POST http://localhost:8081/analyze/anomaly \
  -H "Content-Type: application/json" \
  -d '{
    "metric_name": "test_metric",
    "values": [100, 102, 98, 101, 500],
    "timestamps": ["2026-02-24T10:00:00Z", "2026-02-24T10:01:00Z", "2026-02-24T10:02:00Z", "2026-02-24T10:03:00Z", "2026-02-24T10:04:00Z"],
    "method": "adaptive",
    "tenant_id": "dev-tenant-001"
  }' | jq .
```

---

## 11. Run the Go Gateway

```bash
cd ERP-AIOps/gateway

# Build and run
go run ./cmd/server/

# Run with hot-reload using Air
air

# Run with specific configuration
GATEWAY_PORT=3000 GATEWAY_RUST_API_URL=http://localhost:8080 GATEWAY_AI_BRAIN_URL=http://localhost:8081 go run ./cmd/server/
```

**Verify the Gateway:**

```bash
# Health check (aggregates all backends)
curl -s http://localhost:3000/health | jq .

# Test proxying to Rust API
curl -s http://localhost:3000/api/v1/incidents \
  -H "Authorization: Bearer <dev-token>" \
  -H "X-Tenant-ID: dev-tenant-001" | jq .
```

---

## 12. Start the Frontend

```bash
cd ERP-AIOps/frontend

# Start development server
npm run dev

# The frontend will be available at http://localhost:5173
```

**Verify the Frontend:**

- Open http://localhost:5173 in your browser
- Login with development credentials: `admin@aiops.dev` / `admin123`
- Navigate to the Incidents page to verify API connectivity

---

## 13. Development Workflow

### Daily Development Cycle

```bash
# 1. Start infrastructure (if not already running)
docker compose -f infra/docker-compose.yml up -d yugabytedb dragonflydb rustfs

# 2. Start Rust API with hot-reload (Terminal 1)
cd ERP-AIOps && cargo watch -x 'run --bin aiops-api' -w crates/

# 3. Start AI Brain with hot-reload (Terminal 2)
cd ERP-AIOps/ai-brain && source .venv/bin/activate && uvicorn app.main:app --reload --port 8081

# 4. Start Go Gateway with hot-reload (Terminal 3)
cd ERP-AIOps/gateway && air

# 5. Start Frontend dev server (Terminal 4)
cd ERP-AIOps/frontend && npm run dev
```

### Using Docker Compose for Everything

If you prefer not to run services natively, use Docker Compose for the full stack:

```bash
# Start everything
docker compose -f infra/docker-compose.yml up -d

# View logs
docker compose -f infra/docker-compose.yml logs -f

# Rebuild after code changes
docker compose -f infra/docker-compose.yml build rust-api ai-brain gateway frontend
docker compose -f infra/docker-compose.yml up -d
```

### Running Tests During Development

```bash
# Rust: Run tests on file change
cargo watch -x 'test --workspace' -w crates/

# Rust: Run tests for specific crate
cargo watch -x 'test -p aiops-anomaly' -w crates/aiops-anomaly/

# Python: Run tests on file change
cd ai-brain && ptw tests/ -- -v

# Go: Run tests on file change
cd gateway && air -c .air.test.toml
```

### Database Operations

```bash
# Create a new migration
sqlx migrate add -r {description}
# This creates two files: {timestamp}_{description}.up.sql and {timestamp}_{description}.down.sql

# Run pending migrations
cargo run --bin aiops-migrate -- up

# Rollback the last migration
cargo run --bin aiops-migrate -- down

# Reset database (drop and recreate)
cargo run --bin aiops-migrate -- reset

# Connect to database shell
docker compose -f infra/docker-compose.yml exec yugabytedb ysqlsh -U aiops -d aiops
```

---

## 14. IDE Configuration

### VS Code (Recommended)

**Recommended Extensions:**

| Extension | Purpose |
|-----------|---------|
| `rust-analyzer` | Rust language support (IntelliSense, diagnostics, refactoring) |
| `ms-python.python` | Python language support |
| `ms-python.vscode-pylance` | Python type checking |
| `golang.go` | Go language support |
| `bradlc.vscode-tailwindcss` | Tailwind CSS IntelliSense (if used) |
| `dbaeumer.vscode-eslint` | ESLint integration |
| `esbenp.prettier-vscode` | Prettier formatting |
| `tamasfe.even-better-toml` | TOML file support |
| `redhat.vscode-yaml` | YAML file support |
| `humao.rest-client` | HTTP request testing |

**Workspace Settings (`.vscode/settings.json`):**

```json
{
  "rust-analyzer.check.command": "clippy",
  "rust-analyzer.check.extraArgs": ["--workspace"],
  "rust-analyzer.cargo.features": "all",
  "rust-analyzer.procMacro.enable": true,
  "rust-analyzer.diagnostics.experimental.enable": true,

  "python.defaultInterpreterPath": "${workspaceFolder}/ai-brain/.venv/bin/python",
  "python.analysis.typeCheckingMode": "strict",
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true
  },

  "go.lintTool": "golangci-lint",
  "go.lintFlags": ["--fast"],
  "[go]": {
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    }
  },

  "[rust]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "rust-lang.rust-analyzer"
  },

  "[typescript][typescriptreact]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

### JetBrains IDEs

- **RustRover**: Native Rust support, recommended for Rust-focused work.
- **PyCharm**: For AI Brain development, configure the virtual environment at `ai-brain/.venv`.
- **GoLand**: For Gateway development.
- **WebStorm**: For frontend development.

### Vim/Neovim

- Use `rust-analyzer` LSP for Rust.
- Use `pyright` or `pylsp` for Python.
- Use `gopls` for Go.
- Use `typescript-language-server` for TypeScript.

---

## 15. Troubleshooting

### Rust Build Fails

**Problem:** `cargo build` fails with linking errors.

```bash
# macOS: Install Xcode command line tools
xcode-select --install

# Ubuntu: Install build essentials
sudo apt-get install build-essential pkg-config libssl-dev
```

**Problem:** `sqlx` compile-time query verification fails.

```bash
# Ensure DATABASE_URL is set and the database is running
export DATABASE_URL="postgres://aiops:aiops_dev_password@localhost:5433/aiops"

# Alternatively, use offline mode (uses cached query metadata)
export SQLX_OFFLINE=true
cargo build --workspace
```

### Python Virtual Environment Issues

**Problem:** `pip install` fails for `torch` or `prophet`.

```bash
# Ensure you are using Python 3.12
python3 --version

# macOS: Install system dependencies
brew install cmake openblas

# Ubuntu: Install system dependencies
sudo apt-get install python3.12-dev cmake libopenblas-dev

# Reinstall in a fresh venv
cd ai-brain
rm -rf .venv
python3.12 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip setuptools wheel
pip install -r requirements.txt
```

### Docker Resource Issues

**Problem:** Containers exit with OOMKilled.

```bash
# Increase Docker memory allocation to at least 8GB
# Docker Desktop: Settings > Resources > Memory

# Check current resource usage
docker stats --no-stream
```

### YugabyteDB Connection Refused

**Problem:** Rust API cannot connect to YugabyteDB.

```bash
# Check if YugabyteDB is running
docker compose -f infra/docker-compose.yml ps yugabytedb

# Check YugabyteDB logs
docker compose -f infra/docker-compose.yml logs yugabytedb

# Verify connectivity
docker compose -f infra/docker-compose.yml exec yugabytedb ysqlsh -U aiops -d aiops -c "SELECT 1;"

# If running Rust natively (not in Docker), use localhost
export DATABASE_URL="postgres://aiops:aiops_dev_password@localhost:5433/aiops"
```

### Frontend API Connection Issues

**Problem:** Frontend shows "Network Error" or fails to load data.

```bash
# Verify the Gateway is running
curl -s http://localhost:3000/health | jq .

# Check CORS configuration
curl -v -X OPTIONS http://localhost:3000/api/v1/incidents \
  -H "Origin: http://localhost:5173" \
  -H "Access-Control-Request-Method: GET"

# Verify VITE_API_URL in .env
# Should be: VITE_API_URL=http://localhost:3000
```

### General Reset

If everything is broken, perform a clean reset:

```bash
# Stop all services and remove volumes
docker compose -f infra/docker-compose.yml down -v

# Clean Rust build artifacts
cargo clean

# Recreate Python venv
cd ai-brain && rm -rf .venv && python3.12 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt

# Reinstall frontend dependencies
cd frontend && rm -rf node_modules && npm install

# Start fresh
docker compose -f infra/docker-compose.yml up -d
cargo run --bin aiops-migrate -- up
```

---

*For additional help, reach out on `#erp-aiops-dev` Slack channel or consult the [17-README.md](./17-README.md) quick start guide.*
