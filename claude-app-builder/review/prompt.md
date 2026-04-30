# /review — Standalone Code Review Agent

Use this skill to re-run a review independently (e.g., after manual fixes).

Usage: `/review [feature-name]`

## Process

1. Read `.appbuilder/knowledge-base.md` for project context.
2. Read `.appbuilder/features/[feature-name].md` for the spec.
3. Read `.appbuilder/features/[feature-name]-impl-notes.md` for implementation context.
4. Read `~/.claude/skills/guidelines/frontend.md`, `~/.claude/skills/guidelines/testing.md`, and `~/.claude/skills/guidelines/playwright.md` — use these as the standard to review against.
5. Review ALL changed files related to the feature.

## Review Checklist

**Spec compliance:**
- [ ] All acceptance criteria implemented
- [ ] All `data-testid` attributes present and correctly named per spec
- [ ] All edge cases and error states handled
- [ ] No features added beyond the spec

**Code quality:**
- [ ] TypeScript types correct and complete (no `any`)
- [ ] No obvious security issues (XSS, injection, exposed secrets)
- [ ] Proper error handling at async boundaries
- [ ] No unnecessary re-renders (missing `useMemo`/`useCallback` where it matters)
- [ ] API responses validated before use

**Structure:**
- [ ] Files placed in correct directories per project conventions
- [ ] No dead code or commented-out blocks
- [ ] Imports are clean (no unused imports)

## Output Format

Save to `.appbuilder/features/[feature-name]-review.md`:

```markdown
## Status: APPROVED | CHANGES_REQUIRED

## Blocking Issues
- (list each blocking issue with file:line reference)

## Non-blocking Suggestions
- (list each suggestion)

## Spec Compliance
- [ ] All acceptance criteria met
- [ ] All data-testid attributes present
- [ ] Error states handled
- [ ] No scope creep
```

After saving, report the status to the user. If APPROVED, remind them to run `/unit-test [feature-name]` and `/e2e-test [feature-name]`.
