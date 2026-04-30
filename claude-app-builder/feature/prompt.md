# /feature — Multi-Agent Feature Implementation Pipeline

Orchestrates the full implementation pipeline for a single feature: implement → review → fix → unit test → e2e test.

Usage: `/feature [feature-name]`

## Process

### Step 0 — Load Context

1. Read `.appbuilder/knowledge-base.md` for project context and tech stack.
2. Read `.appbuilder/features/[feature-name].md` for the feature spec.
3. Read `.appbuilder/plan.md` to check dependencies — are all required features marked `done`?

If the spec doesn't exist, list available features from `.appbuilder/features/`.
If dependencies are not met, warn the user and ask if they want to proceed anyway.

Update feature status to `in_progress` in `.appbuilder/plan.md`.

---

### Agent 1 — Implementer

**Role:** Write production-quality code that satisfies the feature spec.

**Instructions:**
- Read the feature spec fully before writing any code.
- Read `~/.claude/skills/guidelines/frontend.md` and apply these rules throughout.
- Follow the project's tech stack from the knowledge base exactly.
- Add `data-testid` attributes to ALL interactive elements and key containers as specified in the feature spec. Do not skip or rename them — E2E tests depend on these exact names.
- Follow existing code conventions in the project (read relevant files first if the project has existing code).
- Write clean, typed code — no `any` in TypeScript, no unhandled promises.
- Do not add features not in the spec. Do not add comments explaining what the code does unless the logic is non-obvious.
- When done, write a brief implementation summary to `.appbuilder/features/[feature-name]-impl-notes.md`.

**Output:** Working code committed to the working branch, implementation notes saved.

---

### Agent 2 — Reviewer

**Role:** Review the implementation against the spec and code quality standards.

**Instructions:**
- Read the feature spec and the implementation notes.
- Read `~/.claude/skills/guidelines/frontend.md`, `~/.claude/skills/guidelines/testing.md`, and `~/.claude/skills/guidelines/playwright.md` — use these as the review standard.
- Review ALL changed files — not just the main component.
- Check against the spec:
  - Are all acceptance criteria met?
  - Are all `data-testid` attributes present and correctly named?
  - Are all edge cases and error states handled?
- Check code quality:
  - TypeScript types correct and complete?
  - No obvious security issues (XSS, injection, exposed secrets)?
  - No unnecessary re-renders or performance pitfalls?
  - Proper error handling at boundaries?
- Write structured feedback to `.appbuilder/features/[feature-name]-review.md`:
  ```
  ## Status: APPROVED | CHANGES_REQUIRED
  
  ## Blocking Issues (must fix before tests)
  - ...
  
  ## Non-blocking Suggestions
  - ...
  
  ## Spec Compliance
  - [ ] All acceptance criteria met
  - [ ] All data-testid attributes present
  - [ ] Error states handled
  ```

**Output:** Review file saved. If `CHANGES_REQUIRED`, Implementer runs again with the review as context.

---

### Agent 1 — Implementer (Fix Round)

Only runs if Reviewer returned `CHANGES_REQUIRED`.

**Instructions:**
- Read `.appbuilder/features/[feature-name]-review.md`.
- Fix ALL blocking issues. Address non-blocking suggestions if they're clearly correct.
- Do not refactor beyond what's needed for the fix.
- After fixing, update the implementation notes.

After fixes, Reviewer runs one more time. If still `CHANGES_REQUIRED` after 2 rounds, pause and ask the user for guidance.

---

### Agent 3 — Unit Tester

**Role:** Write unit and integration tests for the implemented code.

**Instructions:**
- Read `~/.claude/skills/guidelines/testing.md` and apply these rules throughout.
- Use the testing framework from the knowledge base (Jest or Vitest).
- Test each acceptance criterion from the spec with at least one test.
- Cover happy paths, edge cases, and error states listed in the spec.
- Mock external dependencies (API calls, database) at the boundary — do not mock internal modules.
- Use `describe` blocks that match the feature name and acceptance criteria.
- Tests must pass before proceeding. If they don't, fix the test OR flag the issue for Implementer.
- Test files go in `__tests__/` or co-located with source (follow existing project convention).

**Output:** All unit tests passing.

---

### Agent 4 — E2E Tester

**Role:** Write Playwright end-to-end tests using Page Object Model and fixtures.

**Instructions:**
- Read `~/.claude/skills/guidelines/playwright.md` and apply these rules throughout.
- Use Playwright at its latest installed version. Check `package.json` for the exact version.
- Use ONLY `data-testid` selectors — no CSS classes, no XPath, no text matching for interactive elements.
- Structure tests using Page Object Model (POM):
  - Create one Page Object class per page/major component in `e2e/pages/[PageName].page.ts`
  - Page Objects expose semantic methods, not raw selectors: `loginPage.submitForm()` not `page.click('[data-testid="login-form-submit"]')`
- Use Playwright fixtures for:
  - Authenticated user sessions (avoid logging in in every test)
  - Test data setup/teardown
  - Shared page object instances
  - Place fixtures in `e2e/fixtures/`
- Test structure:
  ```
  e2e/
    pages/          ← Page Object classes
    fixtures/       ← Shared fixtures
    tests/          ← Test files grouped by feature
  ```
- Each test file maps to one feature spec.
- Cover: happy path, key error states, and any critical user flows.
- Tests must be independent — no shared state between tests.
- Tests must pass before marking feature done.

**Output:** All E2E tests passing.

---

### Step Final — Mark Done

When all agents complete successfully:
1. Update `.appbuilder/plan.md` — set feature status to `done`.
2. Update `.appbuilder/knowledge-base.md` — add any important decisions or lessons learned to the "Implementation Notes" section.
3. Report to user:
   - Feature implemented and reviewed
   - N unit tests passing
   - N E2E tests passing
   - Next available feature to implement (from plan.md)

## Rules

- Never skip an agent. All 4 must run in order.
- Never mark a feature done if tests are failing.
- If the Implementer needs to make a decision not covered by the spec, pause and ask the user before proceeding.
- Keep the knowledge base up to date — it's the source of truth for future agents.
