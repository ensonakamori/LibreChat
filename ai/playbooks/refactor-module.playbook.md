# Playbook: Refactor a Module

Use this playbook when we want to improve existing code structure without changing behaviour.

## Steps

1. **Understand the current behaviour**
    - Ask me to paste the current module/file.
    - Summarize what it does and how it is used.
    - Identify any tests or call sites you need to see.

2. **Identify refactor goals**
    - Clarify what we’re aiming for, e.g.:
        - better naming,
        - smaller functions,
        - extracting shared logic,
        - removing duplication,
        - isolating Zen-specific logic from LibreChat core.

3. **Propose a refactor plan**
    - Outline 3–6 concrete steps.
    - For each step, mention:
        - affected functions/files,
        - expected risk level.

4. **Apply changes incrementally**
    - Work in small passes, preserving behaviour after each pass.
    - Highlight any places where behaviour might accidentally change.

5. **Review and verify**
    - After refactor:
        - show before/after structure at a high level,
        - point out how it’s now easier to read or extend,
        - suggest tests to confirm behaviour hasn’t changed.

## Notes

- Avoid deep refactors that touch many modules at once unless absolutely necessary.
- If existing code is “good enough”, say so and recommend leaving it alone.
