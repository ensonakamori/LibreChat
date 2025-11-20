# Product Requirements – Zen Studio Command Centre (v1)

## 1. Overview

Zen Studio Command Centre is a LibreChat-based workspace that organizes AI-assisted work around **projects**, a set of **specialist agents**, and **artifacts** such as PRDs and architecture plans.

This document describes the requirements for **v1 (MVP)**.

## 2. Personas

- **Studio Developer / Owner**  
  Uses the command centre daily to run research, draft PRDs, and design architectures for client and internal projects.

- **Collaborator (PM, designer, co-dev)**  
  Accesses specific projects, uses predefined agents, and reads artifacts.

- **Client / Stakeholder (later)**  
  Read-only access to project outputs. v1 only needs to support this implicitly.

## 3. Core Use Cases

1. Create a new project and automatically get the four core flows:
    - Market Research
    - PRD
    - Architecture
    - Implementation Plan

2. Attach conversations and documents to a project, so everything relevant is discoverable in one place.

3. Use specialist agents to:
    - run market research,
    - draft a PRD from research,
    - propose an architecture,
    - create an implementation plan.

4. Export a **project bundle** (JSON + Markdown) for archival, git storage, or migration to another tool.

## 4. Functional Requirements (MVP)

### 4.1 Projects

- Create, edit, list, archive projects.
- Link conversations and artifacts to a `projectId`.
- Show a project overview with:
    - summary,
    - key flows (Research, PRD, Architecture, Plan),
    - artifacts,
    - knowledge base items.

### 4.2 Conversations

- Every conversation may belong to a single project.
- Project view shows only conversations belonging to that project.
- Branching conversations preserve the same `projectId`.

### 4.3 Specialist Agents

- Built-in agents:
    - MarketResearcher
    - PRDWriter
    - Architect
    - ImplementationPlanner
- Project dashboard provides quick actions to start/continue conversations with each agent for that project.

### 4.4 Knowledge Base per Project

- Each project has its own KB:
    - file uploads,
    - URLs.
- Agents automatically use the project KB as context.

### 4.5 Artifacts

- Messages can be saved as Artifacts with:
    - type (research_summary, prd, architecture_plan, task_plan, other),
    - title,
    - link to source message,
    - projectId.
- Project view lists artifacts and provides a clean reading view.

### 4.6 Export

- User can export a project as a zip containing:
    - project metadata,
    - conversations as OpenAI-style JSON,
    - artifacts as Markdown,
    - KB references.

## 5. Non-Functional Requirements

- **Performance**: project dashboard should load within ~2 seconds on typical dev hardware.
- **Reliability**: project data should be durable and backed up via existing LibreChat DB mechanisms.
- **Security**: no additional tracking; API keys remain server-side or in user control.
- **Extensibility**: Zen-specific code should be modular and minimally invasive to LibreChat core.

## 6. Out of Scope (v1)

- Advanced role-based access control.
- Client-facing portal with separate UI.
- Scheduled automation (e.g. nightly research runs).
- Billing, subscriptions, and full SaaS multi-tenant management.
