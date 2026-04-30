# claude-app-builder

Multi-agent Claude Code skills for building web apps — from idea to fully tested production code. Spec-driven, AI-powered SDLC with implement → review → unit test → e2e pipeline.

## How It Works

```
/idea → /plan → /feature [x] → /feature [y] → ... → /status
```

Each `/feature` runs a 4-agent pipeline automatically:

```
Implementer → Reviewer → (fix if needed) → Unit Tester → E2E Tester
```

No manual handoffs. One command = fully implemented, reviewed, and tested feature.

## Skills

| Skill | Description |
|-------|-------------|
| `/idea` | Gather requirements, propose solutions, save to knowledge base |
| `/plan` | Break idea into prioritized, spec-ready features |
| `/feature [name]` | Run full 4-agent pipeline for a single feature |
| `/implement [name]` | Re-run implementation only |
| `/review [name]` | Re-run code review only |
| `/unit-test [name]` | Re-run unit tests only |
| `/e2e-test [name]` | Re-run E2E tests only |
| `/status` | Show current progress across all features |

## Setup

### 1. Clone this repo

```bash
cd ~/Documents/GitHub  # or wherever you keep repos
git clone https://github.com/PiotrLisowski88/claude-app-builder.git
```

### 2. Symlink to Claude Code skills directory

```bash
# Back up existing skills if any
mv ~/.claude/skills ~/.claude/skills.bak 2>/dev/null

# Create symlink
ln -s ~/Documents/GitHub/claude-app-builder ~/.claude/skills
```

### 3. Start building

Navigate to your project directory and run:

```bash
claude
> /idea
```

## Workflow in Detail

### `/idea`
Conversational discovery — Claude asks questions about your app, proposes tech stack, and saves everything to `.appbuilder/knowledge-base.md` in your project.

### `/plan`
Reads the knowledge base and generates individual feature specs in `.appbuilder/features/`. Each spec includes acceptance criteria, API contracts, and a `data-testid` map for every UI element.

### `/feature [name]`
The core skill. Runs 4 agents in sequence:

1. **Implementer** — writes production code following the spec. Adds `data-testid` attributes to all interactive elements as specified.
2. **Reviewer** — checks spec compliance, code quality, security. Returns structured feedback. If issues found, Implementer fixes and Reviewer runs again (max 2 rounds).
3. **Unit Tester** — writes Jest/Vitest tests mapping directly to acceptance criteria. All must pass.
4. **E2E Tester** — writes Playwright tests using Page Object Model and fixtures. Uses only `data-testid` selectors. All must pass.

Feature is marked `done` only when all 4 agents succeed.

### Project Structure (in your app repo)

```
.appbuilder/
  knowledge-base.md     ← project context, stack, decisions
  plan.md               ← feature list with statuses
  features/
    auth.md             ← feature spec
    auth-impl-notes.md  ← implementation notes (auto-generated)
    auth-review.md      ← review feedback (auto-generated)
```

```
e2e/
  pages/                ← Page Object classes
  fixtures/             ← Shared Playwright fixtures
  tests/                ← Test files grouped by feature
playwright.config.ts
```

## Design Decisions

**Why `data-testid` on the Implementer, not the E2E Tester?**
The Implementer knows the component structure and adds `data-testid` as part of the spec. The E2E Tester then writes clean POM selectors against those IDs without touching source code. This enforces a clean separation of concerns.

**Why POM + fixtures for Playwright?**
Page Objects make tests readable and maintainable — `loginPage.login()` is easier to understand and update than raw `page.click('[data-testid="..."]')`. Fixtures avoid login boilerplate in every test.

**Why a max 2-round review loop?**
Infinite loops are a real risk with agentic workflows. After 2 rounds, the skill pauses and surfaces the issue to the user instead of burning tokens in circles.

## Contributing

PRs welcome. If you improve a skill, please test it on a real feature before submitting.
