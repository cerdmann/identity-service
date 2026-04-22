# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install       # Install dependencies
npm start         # Start service on port 3001
```

**CI/CD (GitHub Actions) uses:**
```bash
postman spec lint postman/specs/openapi.yaml   # Lint OpenAPI spec
postman collection run ...                      # Run API tests
```

## Architecture

This is a minimal Node.js/Express identity service — a single file app at `src/index.js`. It is designed to run inside a Kubernetes cluster alongside an `accounts-api` service.

**Data model:** All user data is stored in two in-memory Maps (`usersById`, `usersByEmail`) with no persistence. Data resets on restart.

**Endpoints:**
- `POST /users` — Create user or return existing user by email (upsert semantics)
- `GET /users/:id` — Get user by ID (format: `u1`, `u2`, etc.)
- `GET /users` — List all users
- `GET /health` — Health check

**Deployment:** Docker image runs on Alpine Node.js 20, port 3001. Kubernetes manifests in `k8s/identity.yaml` define the namespace (`identity`), deployment, and service. The K8s manifest includes templated values for Postman Insights API key and project config.

**CI/CD:** Two workflows in `.github/workflows/`:
- `api-testing.yml` — Lints OpenAPI spec and runs Postman collection tests on push/PR
- `sync-to-postman.yml` — Syncs workspace to Postman Cloud on pushes to main
