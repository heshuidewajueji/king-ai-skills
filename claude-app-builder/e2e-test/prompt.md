# /e2e-test — Playwright E2E Test Agent

Use this skill to write or re-run Playwright E2E tests for a feature independently.

Usage: `/e2e-test [feature-name]`

## Process

1. Read `.appbuilder/knowledge-base.md` — check Playwright version and base URL config.
2. Read `.appbuilder/features/[feature-name].md` for acceptance criteria and `data-testid` map.
3. Read `~/.claude/skills/guidelines/playwright.md` — apply these rules throughout.
4. Check existing `e2e/` structure to follow established patterns.

## Project Structure

```
e2e/
  pages/            ← Page Object classes (one per page or major component)
  fixtures/         ← Shared fixtures (auth, test data, page instances)
  tests/            ← Test files grouped by feature
    [feature].spec.ts
playwright.config.ts
```

If this structure doesn't exist yet, create it.

## Page Object Model (POM)

Every page or major component gets a Page Object class:

```typescript
// e2e/pages/LoginPage.ts
import { Page, Locator } from '@playwright/test'

export class LoginPage {
  readonly emailInput: Locator
  readonly passwordInput: Locator
  readonly submitButton: Locator
  readonly errorMessage: Locator

  constructor(private page: Page) {
    this.emailInput    = page.getByTestId('login-form-email')
    this.passwordInput = page.getByTestId('login-form-password')
    this.submitButton  = page.getByTestId('login-form-submit')
    this.errorMessage  = page.getByTestId('login-form-error')
  }

  async goto() {
    await this.page.goto('/login')
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email)
    await this.passwordInput.fill(password)
    await this.submitButton.click()
  }
}
```

**Rules for POM:**
- All locators use `page.getByTestId()` — never CSS classes, IDs, XPath, or text matching for interactive elements.
- Expose semantic methods that describe user actions, not implementation details.
- Page Objects do NOT contain assertions — assertions stay in the test files.
- Use `readonly` Locator properties defined in the constructor.

## Fixtures

Use Playwright fixtures for shared setup:

```typescript
// e2e/fixtures/index.ts
import { test as base } from '@playwright/test'
import { LoginPage } from '../pages/LoginPage'

type Fixtures = {
  loginPage: LoginPage
  authenticatedPage: Page  // pre-logged-in session
}

export const test = base.extend<Fixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page))
  },
  authenticatedPage: async ({ browser }, use) => {
    const context = await browser.newContext({ storageState: 'e2e/.auth/user.json' })
    const page = await context.newPage()
    await use(page)
    await context.close()
  },
})

export { expect } from '@playwright/test'
```

**Authentication fixture:**
- Store authenticated session using `storageState` — do NOT log in in every test.
- Create a global setup script (`e2e/global-setup.ts`) that logs in once and saves state.

## Test File Structure

```typescript
// e2e/tests/login.spec.ts
import { test, expect } from '../fixtures'
import { LoginPage } from '../pages/LoginPage'
import { DashboardPage } from '../pages/DashboardPage'

test.describe('Login Feature', () => {
  test('user can log in with valid credentials', async ({ page }) => {
    const loginPage = new LoginPage(page)
    const dashboardPage = new DashboardPage(page)

    await loginPage.goto()
    await loginPage.login('user@example.com', 'password123')
    
    await expect(page).toHaveURL('/dashboard')
    await expect(dashboardPage.welcomeMessage).toBeVisible()
  })

  test('shows error for invalid credentials', async ({ page }) => {
    const loginPage = new LoginPage(page)

    await loginPage.goto()
    await loginPage.login('user@example.com', 'wrong-password')

    await expect(loginPage.errorMessage).toBeVisible()
    await expect(loginPage.errorMessage).toHaveText(/invalid credentials/i)
  })
})
```

## playwright.config.ts

If it doesn't exist, create it:

```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './e2e/tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'setup', testMatch: /global-setup\.ts/ },
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
      dependencies: ['setup'],
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
})
```

## Selector Rules (strictly enforced)

| Allowed | Not allowed |
|---------|-------------|
| `page.getByTestId('x')` | `.className`, `#id` |
| `page.getByRole('button', { name: '...' })` for semantic elements | `page.locator('button.submit')` |
| `page.getByLabel('Email')` for form fields | `page.locator('form > input:first-child')` |
| | XPath selectors |

Prefer `getByTestId` for all custom components. Use `getByRole` / `getByLabel` only for standard HTML elements where `data-testid` was not added.

## Coverage

For each feature:
- [ ] Happy path (full user journey)
- [ ] Key error states from the spec
- [ ] Any critical branching user flows
- [ ] Mobile viewport if the app is responsive (add a `projects` entry in config)

## Rules

- Tests must be fully independent — no shared state between tests, no order dependency.
- Use `test.beforeEach` for navigation, not for data setup (use fixtures for that).
- Never use `page.waitForTimeout()` — use `expect(locator).toBeVisible()` or `waitForURL()`.
- Run `npx playwright test` after writing and show results.
- If tests fail due to missing `data-testid`, flag it — do NOT change the selector strategy.
