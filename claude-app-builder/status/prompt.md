# /status — Project Status Overview

Shows the current state of all SDLC work in the project.

## Process

1. Read `.appbuilder/knowledge-base.md` for project name and stack.
2. Read `.appbuilder/plan.md` for the feature list and statuses.
3. Check `.appbuilder/features/` for existing specs, review files, and notes.

## Output Format

Print a clear status board:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  [Project Name] — App Builder Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Stack: Next.js 15 · PostgreSQL · Playwright

Features:
  ✅ done        auth           (P0) — login, register, session
  🔄 in_progress dashboard      (P0) — main user view
  ⏳ pending     notifications  (P1) — email + in-app
  ⏳ pending     settings       (P1) — user profile management
  ⏳ pending     export         (P2) — CSV/PDF export

Progress: 1/5 features done (20%)

Next up: /feature dashboard
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If `.appbuilder/` doesn't exist, tell the user to run `/idea` first.
If `plan.md` doesn't exist but `knowledge-base.md` does, tell the user to run `/plan` next.
