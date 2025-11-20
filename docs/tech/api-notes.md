# API Notes

> Draft of planned backend APIs for the Zen Studio layer.

## 1. Project API (Planned)

### `GET /api/projects`

- Returns list of projects current user has access to.

### `POST /api/projects`

- Creates a new project.
- Body: `{ name, slug?, client?, description?, tags? }`.

### `GET /api/projects/:projectId`

- Returns project details, including high-level status.

### `PATCH /api/projects/:projectId`

- Updates project fields.

### `DELETE /api/projects/:projectId`

- Soft-delete or archive project (exact behaviour TBD).

---

## 2. Artifact API (Planned)

### `GET /api/projects/:projectId/artifacts`

- Returns all artifacts for a project.

### `POST /api/projects/:projectId/artifacts`

- Creates a new artifact.
- Body: `{ conversationId, messageId?, type, title, content }`.

### `GET /api/artifacts/:artifactId`

- Returns a single artifact.

### `PATCH /api/artifacts/:artifactId`

- Updates artifact metadata or content.

### `DELETE /api/artifacts/:artifactId`

- Deletes an artifact (or marks as archived).

---

## 3. Export API (Planned)

### `POST /api/projects/:projectId/export`

- Triggers generation of an export bundle.
- Returns:
    - either a direct download,
    - or a link to a stored zip file.

**Bundle contents (target):**

- `project.json`
- `conversations/*.json`
- `artifacts/*.md`
- `kb/*` (files or metadata)
