# Dev Workflow – Solo Dev + AI

This document describes how I, as a solo developer, collaborate with an AI assistant (e.g. Codex/IDE AI) while working on Zen Studio Command Centre.

## 1. Working with Codex / IDE AI

- Use **persistent prompt profiles** from `ai/dev-assistant/`:
    - Pair Programmer
    - Architect
    - Product PM
    - Reviewer
- For each work session:
    1. Pick the appropriate role.
    2. Paste the relevant `.prompt.md` content (or a shortened version).
    3. Paste any necessary context from `docs/` and code files.
    4. Ask for a short plan before generating any code.

## 2. Branching Strategy

- `main`: production-ready branch (tagged for releases).
- `develop`: integration branch for upcoming work.
- `feature/*`: per-feature branches (e.g. `feature/projects-module`).

Workflow:

1. Create `feature/*` branch for each milestone/task.
2. Open PRs into `develop`.
3. Periodically merge `develop` into `main` when stable.

## 3. Commit Message Guidelines

- Use short, descriptive messages:
    - `feat: add Project model and API`
    - `ui: implement project dashboard`
    - `fix: ensure artifacts respect projectId`
- Reference issues when possible:
    - `feat: add project export (closes #12)`

## 4. Milestone-Based Planning

Instead of strict sprints, use **milestones**:

- M1 – Projects & conversations
- M2 – Agents & flows
- M3 – Artifacts
- M4 – KB
- M5 – Export & polish

Before starting a session:

- Choose which milestone/task you’re working on.
- Update `docs/zen-studio-log.md` (or another log) after each session with:
    - what was done,
    - open questions,
    - next steps.
