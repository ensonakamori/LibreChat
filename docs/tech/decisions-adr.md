# Architecture Decision Records (ADRs)

> Lightweight ADRs to track important technical decisions.

---

## ADR-0001: LibreChat Fork vs Separate Service

**Status:** Accepted  
**Date:** YYYY-MM-DD

### Context

Zen Studio Command Centre needs:

- A rich multi-provider chat / agent / RAG foundation.
- A project-centric UX layered on top.
- Minimal infra for a solo dev.

LibreChat already provides a mature open-source base with multi-provider support, agents, MCP, and RAG.

### Decision

We will **fork LibreChat** and implement Zen Studio as:

- A set of additional backend modules (Project, Artifact, Export).
- Frontend components (project sidebar, dashboard, artifacts).
- Minimal schema extensions (e.g. `projectId` on conversations and KB items).

We will try to keep Zen-specific code in dedicated modules to ease future merges from upstream.

### Consequences

- ✅ Faster initial delivery by leveraging existing features.
- ✅ Rich model/provider support “for free”.
- ⚠️ Need to regularly merge updates from upstream and handle conflicts.
- ⚠️ Some refactors may be constrained by LibreChat’s core architecture.

---

## ADR-0002: Project-Scoped Knowledge Base

**Status:** Accepted  
**Date:** YYYY-MM-DD

### Context

Agents should operate in the context of a specific project:

- Different projects have different documents and URLs.
- Mixing KB across projects would create noisy or incorrect responses.

LibreChat already supports a notion of KB / RAG, but not strictly scoped to a project entity.

### Decision

We will **scope KB items and embeddings by `projectId`**:

- Each KB item will store `projectId`.
- RAG queries for a given project will filter by that `projectId`.
- Global/“no project” KB items will be possible but not encouraged in the Zen UX.

### Consequences

- ✅ Cleaner semantics: agents always see project-relevant context.
- ✅ Easier export: project bundle simply collects KB by `projectId`.
- ⚠️ Slightly more complex migration from existing LibreChat instances that use global KB.
- ⚠️ Some future features (global search across projects) will need cross-project queries.
