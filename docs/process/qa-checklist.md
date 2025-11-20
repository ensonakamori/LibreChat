# QA Checklist

> A lightweight set of checks to run before considering a version “done”.

## 1. Manual Checks

### Projects

- [ ] Can create a new project.
- [ ] Project appears in sidebar and can be selected.
- [ ] Project details (name, description, client) display correctly.

### Conversations

- [ ] Creating a conversation from a project attaches the correct `projectId`.
- [ ] Conversations appear in the project view.
- [ ] Branching a conversation keeps the same `projectId`.

### Agents

- [ ] Each built-in agent card on the project dashboard opens the correct conversation/agent.
- [ ] Agents respect project context and KB (when present).

### Artifacts

- [ ] Can save a message as an artifact.
- [ ] Artifact appears in the project’s artifact list.
- [ ] Artifact view displays readable content.

### Export

- [ ] Export Project generates a zip without errors.
- [ ] Bundle contains expected files (project.json, conversations, artifacts).
- [ ] JSON is valid and Markdown renders correctly.

## 2. Edge Cases

- [ ] Behaviour when a project has no conversations.
- [ ] Behaviour when KB is empty.
- [ ] Behaviour when there are many conversations and artifacts (pagination/scroling).
- [ ] Export of a project with no artifacts.
- [ ] Loading project that has been archived/deleted.

## 3. Pre-Release Checklist

- [ ] All critical paths tested (Projects, Agents, Artifacts, Export).
- [ ] No obvious console errors in frontend.
- [ ] API logs show no unhandled exceptions.
- [ ] Environment variables documented and verified in `docs/ops/env-config.md`.
- [ ] CHANGELOG or release notes updated (if used).
