# Unit & Integration Testing Guidelines (Jest / Vitest)

## What to test
- Every acceptance criterion from the feature spec → at least one test
- Edge cases and error states listed in the spec
- Business logic and data transformations
- API response validation and error handling

## What NOT to test
- Implementation details (internal state, private methods)
- Third-party library internals
- Styling or CSS classes
- Things already covered by E2E tests (full user flows)

## Test structure
```typescript
describe('[Feature or Component]', () => {
  describe('[Acceptance criterion or behavior]', () => {
    it('should [expected outcome] when [condition]', () => { ... })
    it('should [error behavior] when [invalid input]', () => { ... })
  })
})
```
- `describe` names map to feature name and acceptance criteria
- `it` names complete the sentence "it should..."
- One logical assertion per test (multiple `expect` calls ok if testing one thing)

## Mocking rules
- Mock at the boundary only: HTTP clients, database clients, auth providers, external SDKs
- Never mock internal utilities, helpers, or modules you own — test them through the component
- Use `msw` (Mock Service Worker) for HTTP mocking when available
- Reset all mocks in `beforeEach`: `jest.clearAllMocks()` / `vi.clearAllMocks()`
- `jest.fn()` / `vi.fn()` for simple stubs; `jest.spyOn` when you need to verify calls

## Async tests
- Always `await` async operations — no floating promises
- Prefer `findBy*` queries (which wait) over `getBy*` + `waitFor` in React Testing Library
- Timeout errors usually mean a missing `await` or an incorrect selector

## React Testing Library
- Query priority: `getByRole` > `getByLabelText` > `getByTestId` > `getByText`
- Use `userEvent` (from `@testing-library/user-event`) not `fireEvent` for user interactions
- Never test internal state — test what the user sees and can do
- Wrap state updates in `act()` only when RTL doesn't do it automatically (rare)

## Coverage
- Don't chase 100% coverage — cover behavior, not lines
- Uncovered lines in error handlers and edge cases are more dangerous than uncovered happy paths
- Run coverage to find gaps, not to hit a number

## Anti-patterns to avoid
- `it.only` / `describe.only` in committed code
- Tests that pass when the implementation is broken (testing mocks, not behavior)
- Shared mutable state between tests — always reset
- Assertions on implementation details (internal variable names, private methods)
- `waitFor` with `getBy*` — use `findBy*` instead
- Empty `catch` blocks in tests — let errors surface
