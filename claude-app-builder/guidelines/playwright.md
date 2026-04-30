# Playwright Guidelines

## Selectors — strict priority order
1. `getByTestId('x')` — always first choice for custom components
2. `getByRole('button', { name: '...' })` — for native HTML elements
3. `getByLabel('Email')` — for form fields with labels
4. Never: CSS classes, IDs, XPath, text matching on interactive elements

## Page Object Model
- One class per page or major component
- Locators defined as `readonly` in constructor, never inline in methods
- Methods describe user actions: `loginPage.submit()` not `page.click(...)`
- No assertions inside Page Objects — assertions belong in test files only
- File: `e2e/pages/[Name].page.ts`

## Fixtures
- Auth session via `storageState` — log in once in global setup, reuse everywhere
- Page Object instances exposed as fixtures — never instantiate in test files directly
- Test data created/cleaned in fixtures, not in `beforeEach`
- File: `e2e/fixtures/index.ts` — re-export custom `test` and `expect` from here

## Test structure
- Each test file = one feature
- Tests are fully independent — no shared mutable state between tests
- `test.beforeEach` for navigation only
- Never `page.waitForTimeout()` — use `expect(locator).toBeVisible()` or `waitForURL()`
- Never `it.only` / `test.only` in committed code

## playwright.config.ts defaults
- `fullyParallel: true`
- `retries: process.env.CI ? 2 : 0`
- `trace: 'on-first-retry'`, `screenshot: 'only-on-failure'`
- `baseURL` from `process.env.BASE_URL || 'http://localhost:3000'`
- `webServer` block with `reuseExistingServer: !process.env.CI`

## Coverage per feature
- Happy path (full user journey)
- Key error states from spec
- Critical branching flows
- Mobile viewport if app is responsive

## Anti-patterns to avoid
- Logging in in every test — use storageState fixture
- Hardcoded `waitForTimeout` — always use explicit waits
- Assertions in Page Objects
- One giant test file for the whole app
- CSS selector fallback when data-testid is missing — flag it, don't work around it
