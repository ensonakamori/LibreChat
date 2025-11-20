# Deployment

This document describes how Zen Studio Command Centre is deployed in different environments.

## 1. Local (Dev)

### Goals

- Fast feedback loop.
- Minimal configuration.

### Typical Setup

- Run services via Docker Compose:
    - App container (LibreChat fork + Zen modules)
    - MongoDB
    - Optional vector DB or search services
- Env vars loaded from `.env.local` or docker compose file.

### Steps (example)

1. Copy `.env.example` to `.env.local` and adjust values.
2. Run `docker compose up` from repo root.
3. Open the app at `http://localhost:3080` (or configured port).

## 2. Staging

### Goals

- Environment resembling production.
- Used for manual QA and experiments.

### Typical Setup

- Same Docker images as production.
- Separate MongoDB and storage bucket.
- Uses staging API keys and test data.
- Deployed to a small VM or container service.

## 3. Production

### Goals

- Stable, secure, monitored environment for real usage.

### Typical Setup

- Containerized deployment (Kubernetes, ECS, or managed containers).
- Managed MongoDB instance with backups.
- S3-compatible object storage for files.
- TLS termination via load balancer or proxy.

### High-Level Steps

1. Build and push Docker image from `main`.
2. Apply infra configuration (Terraform/Helm/… if used).
3. Deploy new version using rolling or blue-green strategy.
4. Monitor logs, performance metrics, and error rates.
