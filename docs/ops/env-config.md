# Environment Configuration

> Placeholder reference for environment variables used by Zen Studio Command Centre.

## 1. LibreChat Base

These are inherited from LibreChat; see upstream documentation for details.

- `MONGO_URI`
- `JWT_SECRET`
- `OPENAI_API_KEY` (and other provider keys)
- `PORT`
- `NODE_ENV`

## 2. Zen Studio Layer (Planned)

Additional variables specific to the Zen layer (names TBD):

- `ZEN_EXPORT_TEMP_DIR` – where export bundles are built before download.
- `ZEN_MAX_EXPORT_SIZE_MB` – safety limit for export bundles.
- `ZEN_DEFAULT_PROJECT_MODEL` – default LLM model per project (optional).

## 3. Secrets Handling

- Store secrets in `.env.local` for local dev (never commit to git).
- In staging/prod:
    - use platform-specific secret managers (e.g. Docker secrets, AWS/GCP secret managers),
    - ensure only app containers have access.

## 4. Configuration Strategy

- Prefer **explicit env vars** for anything that changes between environments.
- Use sensible defaults in code where possible to reduce config noise.
