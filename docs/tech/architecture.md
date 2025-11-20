# Architecture Overview

## 1. Context

Zen Studio Command Centre is a **LibreChat fork/extension** that adds a project-centric layer and studio-specific workflows.

- **Base**: LibreChat (Node.js backend, MongoDB, React frontend).
- **Zen layer**:
    - Project entity and project dashboard.
    - Project-scoped conversations and KB.
    - Specialist agents and artifact management.
    - Export logic for project bundles.

## 2. High-Level Diagram (Text)

Rough component relationships:

- **Frontend**
    - Project Sidebar
    - Project Dashboard
    - Conversation UI (extended with project context)
    - Artifact Viewer
    - KB Management UI

- **Backend**
    - ProjectController / ProjectService
    - ArtifactController / ArtifactService
    - ExportService
    - Existing LibreChat conversation, agent, and KB APIs (extended with `projectId` where needed)

- **Storage**
    - MongoDB collections:
        - `projects`
        - `artifacts`
        - Extended `conversations`
    - Existing vector / KB storage from LibreChat

## 3. Backend Components

- **Project Module**
    - REST endpoints for CRUD on projects.
    - Business logic for linking projects to conversations and artifacts.

- **Artifact Module**
    - API to create, list, update artifacts.
    - Links artifacts to project and underlying conversation.

- **Export Module**
    - Given a `projectId`:
        - fetches metadata, conversations, artifacts, KB references,
        - generates a structured bundle.

- **Integration with LibreChat**
    - Extension of conversation and KB models to store `projectId`.
    - Minimal changes to core where possible (prefer composition/config over deep modification).

## 4. Frontend Components

- **ProjectSidebar**
    - Lists projects, provides project switching and creation.

- **ProjectDashboard**
    - Shows core flows (Research, PRD, Architecture, Plan) as cards.
    - Shows recent artifacts and KB items.

- **ConversationView**
    - Displays chat as in LibreChat, but includes visible project context and quick access to artifacts.

- **ArtifactList & ArtifactView**
    - Lists artifacts for a project and displays them in a focused reading pane.

- **KBManager**
    - Uploads and manages project-scoped documents.

## 5. Data Flow (Project ↔ Conversation ↔ KB ↔ Artifacts)

1. User creates a project.
2. User starts a conversation within that project using a specialist agent.
3. Agent uses project KB and tools to produce responses.
4. Key outputs are saved as artifacts.
5. Export module fetches project + conversations + artifacts + KB references and builds a portable bundle.
