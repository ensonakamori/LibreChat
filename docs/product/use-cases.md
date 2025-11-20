# Key Use Cases

This document captures the main flows Zen Studio Command Centre should support.

## 1. New Project from Scratch

**Scenario:** A new client or internal idea appears.

**Steps (target flow):**

1. User creates a new project with name, client, and short description.
2. System sets up the four core flows (Research, PRD, Architecture, Plan).
3. User optionally uploads initial docs to the project KB.
4. User runs Market Research to explore the idea.

## 2. Market Research

**Scenario:** Understanding competition, users, and risks.

- User opens the project’s Market Research flow.
- MarketResearcher agent:
    - uses web/search tools,
    - produces a structured summary (competitors, ICP, opportunities, risks).
- User refines prompts as needed and saves the final summary as an artifact.

## 3. PRD Creation

**Scenario:** Turning research into a concrete Product Requirements Document.

- From the project dashboard, user starts the PRD flow.
- PRDWriter agent:
    - reads research summaries and KB,
    - outputs a PRD with consistent sections.
- User iterates and saves the PRD as a versioned artifact.

## 4. Architecture & Implementation Planning

**Scenario:** Designing how the product will be built.

- Architect agent proposes stack, architecture, and components.
- ImplementationPlanner agent:
    - breaks architecture into milestones and tasks.
- User exports plan or copies tasks into an issue tracker.

## 5. Exporting a Project Bundle

**Scenario:** Archiving or migrating a project.

- User clicks “Export Project”.
- System generates a zip with:
    - project metadata,
    - conversations as JSON,
    - artifacts as Markdown,
    - KB references.
- User stores bundle in git or another archival system.
