# GitHub Copilot Instructions

## Objective

Provide clear, reproducible, and automatable rules for using GitHub Copilot in this repository: commit style, language, preview, tests, security, and exceptions.

## Scope

- Applies to all changes proposed by Copilot that affect repository source code, tests, or infrastructure.
- Exceptions: product documentation in other languages, specific configuration files (document each exception in the PR).

## General Rules

1. Commits
   - Never use the **"Copilot"** user as the commit author. Commits must always be made using the Git user configured by the responsible person.
   - Recommended commit message convention: `<type>(<scope>): <short summary>` (e.g., `feat(auth): add JWT refresh endpoint`).
   - Common types: `feat`, `fix`, `chore`, `docs`, `test`, `refactor`.

2. Language
   - **Source code and docstrings**: English.
   - **PR messages, previews, and Copilot-to-maintainer communication**: Spanish.
   - Extended documentation may include both languages when appropriate.

3. Code comments
   - Do not add redundant explanatory comments that describe the logic. Explanations should be in the PR preview.
   - Public docstrings (APIs) may be written in English.

4. Mandatory preview
   - Before proposing changes, Copilot must generate a preview with this minimum template:
     - **Summary:** brief description of the change.
     - **Modified files:** short list.
     - **Tests included/affected:** yes/no + summary.
     - **How to validate locally:** commands (e.g. `npm test`, `pytest`).
     - **Known risks / rollback:** brief.
     - **Checklist:** tests ✅, linter ✅, CI ✅.
   - Copilot must not apply non-trivial changes without human approval.

5. Response structure (mandatory)
   - When the response includes steps, options, or scenarios:
     - Be **numbered**.
     - Be **clear and concise** (2–4 lines per step when possible).
     - Follow a **logical order** and prioritize the recommended action.
   - If there are alternatives, list them and mark the recommended option with a brief justification.
   - Example format:
     1. Step 1: Brief action and reason (1–2 lines).
     2. Step 2: Next action and verification command (1–2 lines).
     3. Expected result / exception note (optional).
   - Quick template:
     1. [Brief action] — [Command / check]
     2. [Brief action] — [Command / check]

6. Pull requests and workflow
   - Copilot must propose changes through PRs; direct push to protected branches is not allowed.
   - Minimum PR checklist:
     1. Clear description and issue reference (if applicable).
     2. Included preview (see template).
     3. Tests added/updated when appropriate.
     4. Linters/formatters applied.
     5. CI passing.
     6. Maintainer approval for functional changes.

7. Tests, linters, and hooks
   - Every functional change must include automated tests.
   - Include commands to run tests and linters locally (e.g. `npm test`, `pytest`, `npm run lint`).
   - Use pre-commit hooks (`pre-commit`, `husky`) to run basic checks before commit.

8. Security and dependencies
   - Do not include secrets or credentials in code. Use CI secret-scanning tools.
   - Perform vulnerability scanning when adding dependencies (Dependabot/Snyk or similar).

9. Licenses and third parties
   - Verify license compatibility when copying third-party snippets and document the source in the PR.

10. When to escalate to human review
    - Infrastructure, architecture, security, performance, or changes affecting public contracts require explicit maintainer approval.

11. Compliance automation (CI)
    - CI must validate at minimum:
      - Commit author is not "Copilot".
      - PR includes the mandatory preview.
      - Tests and linters pass (where present).
      - No redundant explanatory code comments are added (where linter applies).

## Templates and examples

### Commit message example

```
feat(auth): add JWT refresh endpoint

- Add route POST /auth/refresh
- Add unit tests for refresh token logic
```

### Minimum preview template

- Summary:
- Modified files:
- Tests included:
- How to validate locally:
  - `npm test` / `pytest`
- Risks and mitigation:
- Checklist:
  - [ ] Tests
  - [ ] Linter
  - [ ] CI passed

### PR checklist example

- [ ] Description and context
- [ ] Preview included
- [ ] Tests added/updated
- [ ] Linters/formatters ok
- [ ] CI successful
- [ ] Maintainer approval (if applicable)

---

These instructions must be applied **to all interactions and code suggestions** made by GitHub Copilot in this repository.
