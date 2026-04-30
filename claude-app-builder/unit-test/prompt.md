# /unit-test — Unit & Integration Test Agent

Use this skill to write or re-run unit tests for a feature independently.

Usage: `/unit-test [feature-name]`

## Process

1. Read `.appbuilder/knowledge-base.md` — check the testing framework (Jest or Vitest).
2. Read `.appbuilder/features/[feature-name].md` for acceptance criteria.
3. Read the implementation files for the feature.
4. Read `~/.claude/skills/guidelines/testing.md` — apply these rules throughout.

## What to Test

Write tests that map directly to acceptance criteria:

```typescript
describe('[Feature Name]', () => {
  describe('[Acceptance Criterion]', () => {
    it('should [expected behavior] when [condition]', () => { ... })
    it('should [error behavior] when [invalid input]', () => { ... })
  })
})
```

**Coverage targets:**
- Every acceptance criterion → at least one test
- Every edge case listed in the spec
- Every error state listed in the spec
- API request/response validation

## Mocking Rules

- Mock external dependencies at the boundary: API calls, database clients, auth providers, third-party SDKs.
- Do NOT mock internal modules, utilities, or helpers — test them through the component under test.
- Use `msw` (Mock Service Worker) for HTTP mocking when available; otherwise use `jest.fn()` / `vi.fn()`.
- Reset all mocks between tests with `beforeEach(() => { jest.clearAllMocks() })`.

## File Placement

Follow existing project convention:
- Co-located: `[component].test.ts` next to `[component].ts`
- Centralized: `__tests__/[feature-name]/`

If no convention exists, use co-located.

## Rules

- Tests must be deterministic — no random data without seeded generators.
- No `it.only` or `describe.only` left in final tests.
- All tests must pass before reporting done.
- If a test reveals a bug, report it to the user — do not silently fix the implementation.
- Run the test suite after writing: `npm test` / `npx vitest run` and show results.
