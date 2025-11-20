# Playbook: Write Tests

Use this when adding tests for new or existing functionality.

## Steps

1. **Clarify the behaviour to test**
    - Ask for:
        - the relevant code,
        - any requirements from PRD / docs,
        - example inputs/outputs if I have them.

2. **Identify test cases**
    - List:
        - happy path scenarios,
        - important edge cases,
        - obvious failure modes.

3. **Propose test structure**
    - Decide:
        - test framework (based on repo),
        - where tests should live (paths, filenames),
        - naming conventions.

4. **Write test skeletons**
    - Generate test files with:
        - descriptive test names,
        - clear arrange–act–assert structure,
        - TODO comments if needed.

5. **Refine and extend**
    - After I run tests and provide feedback:
        - fix any issues,
        - add missing coverage.

## Notes

- Focus tests on behaviour, not implementation details.
- When mocking or stubbing is needed, explain why and how.
