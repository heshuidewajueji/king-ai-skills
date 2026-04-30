# write-unit-tests

A Claude Code skill that writes unit tests matching your codebase's existing patterns — without assuming the stack.

## What it does

The most common failure mode when an AI writes tests is assuming the framework, fixture style, and assertion patterns rather than reading them. This skill fixes that: it reads the git diff to identify what changed, discovers your test framework and existing test patterns from the codebase, and then writes new tests that mirror them exactly — same naming, same fixture usage, same assertion style, same file location. Coverage checklist included so nothing gets skipped.

## Installation

Copy the skill folder into your Claude Code skills directory:

```bash
cp -r write-unit-tests ~/.claude/skills/
```

Then invoke it in Claude Code with `/write-unit-tests`.

## Usage

Run `/write-unit-tests` after making changes to any feature or function. The skill will:
1. Read the git diff to identify what needs coverage
2. Discover your test framework (`pytest`, `vitest`, `jest`, etc.) from config files
3. Read 1-2 existing test files to learn naming and fixture conventions
4. Write tests covering happy path, error paths, boundary values, and auth failures
5. Run the test suite to confirm everything passes

Works with Python (`pytest`), TypeScript/JavaScript (`jest`, `vitest`), and any other framework discoverable from project config.

## Requirements

- A git repo with at least one prior commit (uses `git diff` to find changes)
- No env vars or API keys needed

## License

MIT
