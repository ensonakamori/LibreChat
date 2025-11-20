**Product Requirements Document (PRD)** for the **Zen Studio Command Centre** built as a LibreChat remake.

---

# Zen Studio Command Centre – Product Requirements Document

## 1. Product Overview

### 1.1 Vision

Create a **calm, focused, AI-first workspace** for a small digital studio where all project work (market research, PRDs, architecture, implementation planning) happens through carefully designed **specialist AI agents** inside a unified, minimalist UI.

Instead of a messy pile of chats, the system organizes everything around **projects**, **agents**, and **artifacts** (documents, plans, diagrams) – so it feels like a studio command centre, not “yet another chat app”.

### 1.2 One-line elevator pitch

> A Zen-like, opinionated AI command centre for product & web projects, built on LibreChat, with project-centric views and a small team of specialist agents handling research, PRDs, and architecture.

### 1.3 Success criteria (first phase)

* You personally can:

    * Run the app locally or self-host.
    * Manage all studio projects in it (no more scattered chats across tools).
    * Generate **consistent, reusable outputs**: research briefs, PRDs, architecture plans, task plans.
* The system is:

    * Simple enough to demo to a client or another dev without explaining “what LibreChat is”.
    * Cleanly separated from LibreChat core so you can pull upstream updates without huge pain.

---

## 2. Background & Motivation

Current AI tools focus on **ad-hoc chatting**. For a studio:

* Conversations are scattered (ChatGPT, local UIs, browser tabs).
* No strong concept of **project/workspace**, just chat histories.
* Reusing structured outputs (PRDs, plans) is manual and fragile.

You want:

* A **single place** to run all AI-assisted work for a project.
* A set of **reliable, specialized agents** that act like “virtual colleagues”.
* Clean **exports** so your work is not locked in.

LibreChat offers a strong base (multi-provider, agents, MCP, RAG, MIT license), but needs:

* A more opinionated *studio* UX.
* Project-level organization.
* A curated set of specialist agents.
* Better export/migration story.

---

## 3. Target Users & Personas

### 3.1 Primary persona – Studio Developer/Owner

* Role: freelance/agency dev, designer–developer, tech lead.
* Needs:

    * Centralize all project AI work.
    * Quickly move from idea → research → PRD → architecture → plan.
    * Maintain calm, uncluttered environment.

### 3.2 Secondary persona – Collaborator (PM, designer, co-dev)

* Role: collaborator invited to work in the studio command centre.
* Needs:

    * Access specific projects.
    * Use predefined agents.
    * View and comment on artifacts and outputs.

### 3.3 Tertiary persona – Client (read-only)

* Role: external client or stakeholder.
* Needs:

    * View project space (research summaries, PRD, architecture plan).
    * Possibly comment or ask questions (optional later).

---

## 4. Core Use Cases

### 4.1 Create a new project and set up AI workspace

* User creates a **Project** (e.g. “Acme SaaS Dashboard”).
* System auto-creates:

    * “Market Research” conversation with `MarketResearcher`.
    * “Product Requirements” conversation with `PRDWriter`.
    * “Architecture Plan” conversation with `Architect`.
    * “Implementation Plan” conversation with `ImplementationPlanner`.
* Project homepage shows the four stages as cards with status.

### 4.2 Run market research for a new product idea

* User opens Project → “Market Research”.
* Sends a brief (“B2B SaaS, small companies, feature X”).
* `MarketResearcher`:

    * runs web/search tools,
    * outputs structured findings: competitors, target users, pains, opportunities.
* User can pin/save key findings as artifacts.

### 4.3 Generate a PRD from research

* From the Project view, user clicks “Draft PRD”.
* System:

    * loads context from research convo + KB docs,
    * prompts `PRDWriter` to output PRD in a predefined schema.
* PRD is saved as a **versioned artifact** tied to the project.

### 4.4 Create architecture and implementation plan

* User opens “Architecture Plan”.
* `Architect`:

    * proposes architecture stack, services, data model, folder structure.
* User then opens “Implementation Plan”.
* `ImplementationPlanner`:

    * breaks architecture into milestones, tasks, and rough timelines.

### 4.5 Export project bundle

* User clicks “Export Project”.
* System generates:

    * JSON for all conversations related to this project (OpenAI messages format).
    * Markdown files for key artifacts (research summary, PRD, architecture, plan).
* Zips into a folder (`project-slug_YYYYMMDD.zip`) for storage in git or elsewhere.

---

## 5. Product Scope

### 5.1 In scope (MVP + near future)

* Project/workspace layer on top of LibreChat.
* A curated set of 3–4 specialist agents.
* Project-centric UI (sidebar with projects, project dashboard).
* Knowledge base per project (KB docs + RAG for agents).
* Project export (JSON + Markdown).
* Simple multi-user support (reusing LibreChat’s auth, but with project permissions).

### 5.2 Out of scope (for now)

* Full multi-tenant SaaS admin panel and billing.
* Complex role hierarchies (e.g. granular per-conversation ACLs).
* Visual drawing tools (diagrams etc.) beyond what artifacts provide.
* Heavy automation (scheduled runs / CRON agents) in v1.

---

## 6. Functional Requirements

### 6.1 Projects / Studio Spaces

**FR-1** – Create Project

* User can create a project with: `name`, `slug`, `client`, `description`, `tags`.
* Optional: select default model + temperature profile.

**FR-2** – Project listing

* Sidebar lists projects with:

    * Name, client, small status indicator (e.g. PRD done / not done).
* Projects can be searched and filtered (by tag, client).

**FR-3** – Project overview page

* For a selected project, show:

    * Summary (name, client, description).
    * Key agents & their main conversations (Research, PRD, Architecture, Plan).
    * Last updated timestamps.
    * Attached KB docs.
    * Key artifacts (pinned).

**FR-4** – Attach conversations to a project

* Every conversation has a `projectId`.
* Creating a new conversation from inside the project sets `projectId` automatically.
* Existing conversations can be reassigned to projects via UI.

---

### 6.2 Conversations & Chat

**FR-5** – Conversation model

* Conversation must support:

    * link to project (`projectId`),
    * link to default agent (`agentId`),
    * model configuration overrides (if needed).

**FR-6** – Conversation list (per project)

* Within project view, show only conversations belonging to that project.
* Support conversation pinning, renaming, archiving.

**FR-7** – Branching & history

* Keep LibreChat’s branching feature (fork at any turn).
* Branches preserve `projectId`.

---

### 6.3 Specialist Agents

You will define and ship **built-in agents**:

**FR-8** – Market Researcher agent

* System prompt template includes:

    * domain (SaaS / web products),
    * instructions for competitor analysis, positioning, SWOT,
    * constraints: structured sections (Overview, Competitors, ICP, Opportunities, Risks).
* Tools:

    * Web search / MCP SERP,
    * Possibly scraping.

**FR-9** – PRD Writer agent

* System prompt instructs to output a PRD in **consistent schema**:

    * Context, Goals, Target Users, User Stories, Scope, Non-Scope, Risks, Open Questions.
* Uses:

    * project KB,
    * previous research conversation(s).

**FR-10** – Architect agent

* System prompt:

    * focus on web app architecture (Next.js, APIs, DB, auth),
    * consistent structure: high-level architecture, components, data model, deployment, tech choices.
* Optionally uses:

    * your “studio architecture standards” as KB docs.

**FR-11** – Implementation Planner agent

* System prompt:

    * output milestones and tasks, grouped by phases (MVP, v1, v2),
    * each task includes: description, dependencies, rough estimate.
* Uses architecture output as input context.

**FR-12** – Agent presets & visibility

* These four agents must:

    * appear prominently in the “Project actions” UI.
    * be protected from accidental deletion.
* Regular LibreChat agents remain available via advanced settings, but are secondary.

---

### 6.4 Knowledge Base (KB) per Project

**FR-13** – Project KB

* Each project has its own KB:

    * file uploads (pdf, md, txt, docx),
    * URLs (for scraping & indexing).
* KB is used by all agents bound to that project.

**FR-14** – KB management UI

* In project view, show:

    * list of KB items (file name, type, date added).
* Actions:

    * upload, delete, resync, reindex.

**FR-15** – Agent integration with KB

* When an agent runs within a project:

    * the RAG pipeline automatically queries that project’s KB.
* There should be a visible indication when KB context is used (optional small badge / note).

---

### 6.5 Artifacts (Documents / Outputs)

**FR-16** – Artifacts as first-class items

* Ability to save a message (or generated content) as an **Artifact** with:

    * `title`, `type` (research_summary, prd, architecture_plan, task_plan, other),
    * link to source message / conversation,
    * `projectId`.

**FR-17** – Artifact view

* For each project, show artifacts in a “Documents” area.
* Clicking an artifact opens:

    * a clean reading view (Markdown/HTML),
    * link back to original conversation.

**FR-18** – Versioning

* When saving an artifact of the same type for the same project:

    * either create versions or keep a simple “Latest / Previous versions” list.

---

### 6.6 Command Centre UI

**FR-19** – Minimal navigation

* Primary nav sections:

    1. Projects
    2. Agents (list/presets)
    3. Settings
* Hide or de-emphasize raw provider playgrounds unless in “Advanced mode”.

**FR-20** – Project dashboard layout

* Top: project summary.
* Middle: 4 core flows as cards:

    * Market Research
    * PRD
    * Architecture
    * Implementation Plan
* Each card displays:

    * status (Not started / In progress / Done),
    * last updated, link to conversation.
* Bottom: artifacts list & KB docs.

**FR-21** – Quick actions

* Buttons like:

    * “Run Market Research”
    * “Draft PRD from Research”
    * “Update Architecture”
    * “Regenerate Plan”
* They open (or continue) corresponding conversations with correct agent & project context.

---

### 6.7 Models & Providers Configuration

**FR-22** – Curated model set

* In default config, only show a small curated list in the UI (e.g. `gpt-4.1`, `gpt-4o-mini`, one local model).
* Advanced users can enable “Show all models” in Settings.

**FR-23** – Per-project default model

* Each project can optionally override the default model/temperature.
* Agents in that project use those defaults unless explicitly changed.

---

### 6.8 Export & Import

**FR-24** – Export project bundle

* From project view: “Export Project” → downloads:

    * `/project.json` – metadata, list of conversations & artifacts,
    * `/conversations/*.json` – messages in OpenAI `messages` format,
    * `/artifacts/*.md` – Markdown documents,
    * `/kb/*` – original docs or references.

**FR-25** – Import project (later phase)

* Ability to import from a bundle:

    * re-create project,
    * conversations,
    * KB references.

---

### 6.9 Multi-user & Permissions (MVP)

**FR-26** – Use LibreChat auth

* Keep LibreChat’s existing auth (email/OAuth) as-is.

**FR-27** – Project ownership

* Projects have `ownerId` and an optional `sharedWith` list.
* Only owners can delete or export a project.
* Shared users can:

    * view project,
    * create/edit conversations under that project.

(Advanced ACL roles can be considered later.)

---

## 7. Non-Functional Requirements

### 7.1 Performance

* Initial project dashboard load < 2 seconds on typical dev hardware.
* Chat streaming latency mostly depends on model provider; UI should show partial tokens as they arrive.

### 7.2 Reliability & Backup

* Support standard DB backup strategies (LibreChat’s DB).
* Export feature acts as an additional backup for project outputs.

### 7.3 Security & Privacy

* No collection of telemetry beyond what LibreChat already uses unless explicitly added.
* All API keys stored server-side (env vars).
* HTTPS/TLS assumed in deployment (not handled by app but documented).

### 7.4 Extensibility

* Avoid hardcoding things deep in LibreChat core:

    * Put project & studio logic in separate “studio” modules where possible.
* Clearly separate:

    * upstream LibreChat code,
    * your Zen Studio customizations.

---

## 8. Technical Architecture (High Level)

### 8.1 Base

* **Backend**: LibreChat’s Node.js + MongoDB + whatever search/vector stack it already employs.
* **Frontend**: LibreChat React app.

### 8.2 Zen Studio modules (ideal structure)

* `backend/studio/`

    * `models/Project.ts`
    * `controllers/ProjectController.ts`
    * `routes/studioProjects.ts`
    * `services/ExportService.ts`
* `frontend/studio/`

    * `pages/ProjectsPage.tsx`
    * `pages/ProjectDetailPage.tsx`
    * `components/ProjectSidebar.tsx`
    * `components/ArtifactList.tsx`
    * `components/StudioQuickActions.tsx`

### 8.3 Integration with LibreChat

* Extend existing conversation schema to include `projectId`.
* Extend agent config to mark “built-in studio agents”.
* Wrap existing KB / RAG APIs with project scoping.

---

## 9. Phased Implementation Plan

### Phase 1 – Skeleton

* Fork LibreChat.
* Add `Project` model & basic CRUD API.
* Add simple Projects sidebar + detail page.
* Attach conversations to projects.

### Phase 2 – Specialist agents

* Define `MarketResearcher`, `PRDWriter`, `Architect`, `ImplementationPlanner` in agents config/UI.
* Wire quick actions from project dashboard to these agents.

### Phase 3 – Artifacts & documents

* Add `Artifact` model & API.
* UI to save messages as artifacts and view them in project.

### Phase 4 – Knowledge base per project

* Wire existing KB/RAG to be project-scoped.
* Add simple KB management UI on project page.

### Phase 5 – Export bundles

* Implement `ExportService` and “Export Project” button.
* Verify you can reconstruct project context from the exported bundle.

### Phase 6 – Polish & Zen UX

* Simplify navigation into 3 main areas.
* Tweak theme, typography, spacing.
* Hide “power features” behind an Advanced switch.

---

If you’d like, next I can drill into **one section as implementation-ready detail** (for example, define the exact **TypeScript interfaces** for `Project`, `Artifact`, and how to extend LibreChat’s `Conversation` model).
