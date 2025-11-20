Nice, this is exactly the kind of thing that’s worth doing once and reusing forever.

Below is a **single, comprehensive prompt** you can paste into **Codex CLI** (or your IDE’s AI) to have it set up the folder + file structure and populate each file with sensible skeleton content.

I’ll put it in a code block so you can copy it as-is.

---

````text
You are Codex CLI, connected to my local repo and able to create/edit files.

## High-level context

I’m building **Zen Studio Command Centre**, a LibreChat-based “AI command centre” for my studio.  
I want a clean structure for:

- Human-facing docs (product + technical + ops)
- Prompts I’ll use with my IDE AI (dev assistant)
- Prompts that will be used INSIDE the app (agents, workflows, system style)

Your job in this session: **create the folder + file skeleton and populate each file with minimal but meaningful starter content**.

Assume the current working directory is **the root of my LibreChat fork**.

---

## General rules

1. **Do not modify existing source files** unless absolutely necessary. This task is only about *creating new folders/files*.
2. Use **POSIX-friendly paths**.
3. For each file:
   - Add a short header explaining its purpose.
   - Use Markdown for `.md` files.
   - Use Markdown-style sections inside `.prompt.md` as well.
4. If you cannot directly create files, instead:
   - Output a list of shell commands (`mkdir`, `cat <<'EOF' > file`) I can run to reproduce exactly what you describe.

---

## 1. Create `docs/` structure

Create these folders:

- `docs/product/`
- `docs/tech/`
- `docs/process/`
- `docs/ops/`

Create and populate the files as follows:

### 1.1 docs/product/

**`docs/product/vision.md`**

Content skeleton:

- H1: “Zen Studio Command Centre – Vision”
- Sections:
  - Problem
  - Solution
  - Target Users
  - Positioning & Differentiation
  - High-level Goals

Add 1–2 placeholder sentences under each heading.

---

**`docs/product/prd-zen-studio-v1.md`**

Content skeleton:

- H1: “Product Requirements – Zen Studio Command Centre (v1)”
- Sections:
  - Overview
  - Personas
  - Core Use Cases
  - Functional Requirements (high-level list)
  - Non-Functional Requirements
  - Out of Scope (v1)

Add short placeholder bullets in each section; I’ll later replace them with my detailed PRD.

---

**`docs/product/roadmap.md`**

Content skeleton:

- H1: “Roadmap”
- Sections:
  - MVP (v1)
  - v1.1 – UX improvements
  - v2 – Automation & Integrations

Each section: bullet list with `TODO` placeholders.

---

**`docs/product/use-cases.md`**

Content skeleton:

- H1: “Key Use Cases”
- Sub-sections for:
  - New project from scratch
  - Market research
  - PRD creation
  - Architecture & implementation planning
  - Exporting project bundle

Each sub-section: brief description + `TODO: flesh out steps`.

---

### 1.2 docs/tech/

**`docs/tech/architecture.md`**

Content skeleton:

- H1: “Architecture Overview”
- Sections:
  - Context (LibreChat fork + Zen Studio layer)
  - High-level Diagram (text description for now)
  - Backend Components
  - Frontend Components
  - Data Flow (Project ↔ Conversation ↔ KB ↔ Artifacts)

Use bullet points to outline how the Zen Studio layer sits on top of LibreChat.

---

**`docs/tech/data-model.md`**

Content skeleton:

- H1: “Data Model”
- Sections:
  - Project
  - Artifact
  - Conversation (extended)
  - Knowledge Base / RAG

Under each, create a TypeScript-style interface in a fenced code block, *as a starting point*, e.g.:

```ts
interface Project {
  id: string;
  name: string;
  slug: string;
  client?: string;
  description?: string;
  tags?: string[];
  createdAt: string;
  updatedAt: string;
}
````

(Be explicit but keep fields reasonably minimal; I’ll refine later.)

---

**`docs/tech/decisions-adr.md`**

Content skeleton:

* H1: “Architecture Decision Records (ADRs)”
* Add ADR templates for at least 2 example decisions:

    * ADR-0001: LibreChat fork vs separate service
    * ADR-0002: Project-scoped KB approach

Use a standard ADR structure (Context, Decision, Consequences) with placeholders.

---

**`docs/tech/api-notes.md`**

Content skeleton:

* H1: “API Notes”
* Sections:

    * Project API (planned endpoints)
    * Artifact API (planned endpoints)
    * Export API

Just outline endpoints and HTTP verbs in bullet form for now.

---

### 1.3 docs/process/

**`docs/process/dev-workflow.md`**

Content skeleton:

* H1: “Dev Workflow – Solo Dev + AI”
* Sections:

    * Working with Codex / IDE AI
    * Branching Strategy
    * Commit Message Guidelines
    * Milestone-based Planning

Short paragraphs + bullets; include a mini checklist for starting a work session.

---

**`docs/process/qa-checklist.md`**

Content skeleton:

* H1: “QA Checklist”
* Sections:

    * Manual Checks (projects, agents, artifacts, export)
    * Edge Cases
    * Pre-release Checklist

Use checkbox-style lists (`- [ ]`) for items.

---

### 1.4 docs/ops/

**`docs/ops/deployment.md`**

Content skeleton:

* H1: “Deployment”
* Sections:

    * Local (dev)
    * Staging
    * Production

For each: bullet list of steps; mention Docker/Compose as the likely approach.

---

**`docs/ops/env-config.md`**

Content skeleton:

* H1: “Environment Configuration”
* Sections:

    * Required env vars (LibreChat base)
    * Additional env vars (Zen Studio layer)
    * Secrets handling notes

Just list placeholder env names; I’ll fill details.

---

## 2. Create `ai/` structure (for dev assistant prompts)

Create:

* `ai/dev-assistant/`
* `ai/playbooks/`
* `ai/notes/`

### 2.1 ai/dev-assistant/

Each of these `.prompt.md` files should:

* Start with `# Role`, `# Context`, `# Behaviour`, `# Output style`.

**`ai/dev-assistant/pair-programmer.prompt.md`**

* Role: Pair programmer for Zen Studio Command Centre.
* Context: LibreChat fork; docs in `docs/`.
* Behaviour: ask for context, suggest minimal diffs, explain trade-offs.

**`ai/dev-assistant/architect.prompt.md`**

* Role: Software architect.
* Behaviour: focus on module boundaries, data model, impact on existing LibreChat code.

**`ai/dev-assistant/product-pm.prompt.md`**

* Role: Product manager.
* Behaviour: help refine PRD, prioritize scope, generate user stories.

**`ai/dev-assistant/reviewer.prompt.md`**

* Role: code reviewer.
* Behaviour: comment on readability, consistency, potential bugs.

Populate each with 1–2 paragraphs + bullet points per section.

---

### 2.2 ai/playbooks/

Each `.playbook.md` describes a repeatable collaboration flow with the AI.

**`ai/playbooks/implement-feature.playbook.md`**

Sections:

* Title
* Steps (1..N)
* Notes

Steps should define a standard process: restate task, plan steps, request code, propose diffs, summarize.

**`ai/playbooks/refactor-module.playbook.md`**

Similar structure, but focused on refactoring.

**`ai/playbooks/write-tests.playbook.md`**

Focused on writing unit/integration tests, including identifying test cases from requirements.

---

### 2.3 ai/notes/

**`ai/notes/session-log.md`**

* H1: “AI Session Log”
* A table with columns: Date, Agent Role, Topic, Notes.
* Add one example row with placeholder values.

---

## 3. Create `prompts/` structure (for in-app agents)

Create:

* `prompts/agents/`
* `prompts/workflows/`
* `prompts/system/`

### 3.1 prompts/agents/

Each agent prompt should include:

* `# Role`
* `# Goal`
* `# Output Structure`
* `# Style`
* `# Inputs`
* `# Constraints`

**`prompts/agents/market-researcher.prompt.md`**

* Focus: competitor & market analysis for digital products.
* Output structure: Overview, Competitors, ICP, Opportunities, Risks.

**`prompts/agents/prd-writer.prompt.md`**

* Focus: writing PRDs from research + brief.
* Output structure: Context, Goals, Users, User Stories, Scope, Out-of-Scope, Risks, Open Questions.

**`prompts/agents/architect.prompt.md`**

* Focus: web app architecture (Next.js + API + DB).
* Output: Architecture Overview, Components, Data Model, Deployment, Trade-offs.

**`prompts/agents/implementation-planner.prompt.md`**

* Focus: milestones & tasks.
* Output: Phases, Milestones, Tasks (with dependencies and rough estimates).

Populate each with 1–3 bullets under each section.

---

### 3.2 prompts/workflows/

**`prompts/workflows/project-kickoff.prompt.md`**

* Purpose: Guide an agent through initial project kickoff (understanding project, clarifying goals, identifying unknowns).

**`prompts/workflows/project-retro.prompt.md`**

* Purpose: Reflect on a completed project or milestone, extract lessons, improve process.

Each should have sections: Role, Steps, Expected Output.

---

### 3.3 prompts/system/

**`prompts/system/global-style.prompt.md`**

* Describe general style guidelines for all agents:

    * concise but structured,
    * prefer numbered lists when helpful,
    * explicitly mark TODOs and open questions,
    * maintain a calm, “Zen” tone.

---

## Final instruction

1. Create all directories and files described above.
2. Populate each file with the specified headers and placeholder content.
3. If you cannot directly create files, output a bash script or a sequence of shell commands that I can run to create them with the desired contents.

When you’re done, show me:

* A tree view of the new directories and files.
* A short note if any step had to be adjusted because of the existing repo structure.

```

---

You can tweak any wording, but if you paste this into Codex CLI from the repo root, it should either:

- directly create the structure, or  
- give you a ready-to-run shell script to do it.
::contentReference[oaicite:0]{index=0}
```
