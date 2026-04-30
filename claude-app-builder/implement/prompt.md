# /implement — Standalone Implementation Agent

Use this skill when you want to re-run implementation only (e.g., after manual spec changes) without going through the full `/feature` pipeline.

Usage: `/implement [feature-name]`

## Process

1. Read `.appbuilder/knowledge-base.md` for project context and tech stack.
2. Read `.appbuilder/features/[feature-name].md` for the spec.
3. Read `.appbuilder/features/[feature-name]-review.md` if it exists — incorporate previous review feedback.
4. Read `~/.claude/skills/guidelines/frontend.md` — apply these rules throughout implementation.
5. Implement the feature following the same rules as Agent 1 in `/feature`.

## Rules

- Add `data-testid` to ALL interactive elements per the spec.
- Follow the project's existing code conventions.
- No TypeScript `any`, no unhandled promises.
- Write implementation notes to `.appbuilder/features/[feature-name]-impl-notes.md`.
- After implementation, remind the user to run `/review [feature-name]` next.
