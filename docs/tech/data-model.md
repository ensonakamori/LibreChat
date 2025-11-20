# Data Model

## Project
Represents a single initiative with metadata used to scope chats and artifacts.

```ts
interface Project {
  id: string;
  name: string;
  slug: string;
  client?: string;
  description?: string;
  tags?: string[];
  status?: "draft" | "active" | "archived";
  createdAt: string;
  updatedAt: string;
}
```

## Artifact
Stores outputs and references linked to a project.

```ts
interface Artifact {
  id: string;
  projectId: string;
  type: "note" | "research" | "prd" | "plan" | "export" | "other";
  title: string;
  summary?: string;
  contentUri?: string;
  sourceConversationId?: string;
  createdAt: string;
  updatedAt: string;
}
```

## Conversation (extended)
Extends LibreChat conversations with project awareness.

```ts
interface ProjectConversation {
  id: string;
  projectId?: string;
  title?: string;
  participantIds?: string[];
  lastMessageAt: string;
  artifactIds?: string[];
}
```

## Knowledge Base / RAG
Represents ingested knowledge scoped to a project.

```ts
interface KnowledgeBaseEntry {
  id: string;
  projectId: string;
  source: "upload" | "url" | "note" | "repo";
  label: string;
  chunkCount?: number;
  embeddingStatus: "pending" | "complete" | "failed";
  createdAt: string;
}
```
