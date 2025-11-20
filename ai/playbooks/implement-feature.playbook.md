# Playbook: Implement a Feature

This playbook defines how I want to collaborate with the AI assistant when implementing a new feature.

## Steps

1. **Clarify the task**
    - Restate the feature in your own words.
    - Ask me for links/snippets from:
        - `docs/product/prd-zen-studio-v1.md`
        - `docs/tech/architecture.md`
        - `docs/tech/data-model.md`
        - any relevant code files.

2. **Propose a mini-plan**
    - Outline a plan of at most **5–8 steps**.
    - Call out any risky or ambiguous parts.

3. **Work file-by-file**
    - For each step:
        - Ask for the current content of the file(s) if you don’t have it.
        - Propose **minimal changes** or patch-style snippets.
    - Prefer integrating with existing patterns over new abstractions.

4. **Keep me in the loop**
    - After each chunk of changes, briefly explain:
        - what changed,
        - why it’s safe,
        - what might break.

5. **Wrap up**
    - Summarize all changes in bullet points.
    - Suggest a short list of manual tests.
    - If applicable, propose follow-up TODOs for later refactors.

## Notes

- If you are unsure about the existing architecture or constraints, ask me to paste the relevant docs instead of making assumptions.
- When there is more than one reasonable approach, offer options with pros/cons.
