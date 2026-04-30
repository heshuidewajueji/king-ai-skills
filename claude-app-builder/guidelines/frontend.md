# Frontend Guidelines (React / Next.js / TypeScript)

## TypeScript
- No `any` — use `unknown` and narrow, or define a proper type
- No type assertions (`as Foo`) unless you can't avoid it — add a comment if you do
- Prefer `type` over `interface` for props; use `interface` for extendable contracts
- Return types on exported functions — infer internally
- Validate external data at boundaries (API responses, form input) — never trust raw JSON

## React components
- One component per file; filename matches component name
- Props destructured in signature, not inside body
- No inline object/array literals in JSX (causes re-renders): extract as constants or useMemo
- `key` prop must be stable and unique — never array index for dynamic lists
- Avoid prop drilling beyond 2 levels — use context or co-locate state

## State management
- Local state first (`useState`) — lift only when actually shared
- Server state (API data) via React Query / TanStack Query — no manual fetch in `useEffect`
- `useEffect` only for external side effects (subscriptions, DOM APIs) — not for derived state
- Derived values: compute inline or `useMemo`, don't store in state

## Next.js (App Router)
- Server Components by default — add `'use client'` only when you need interactivity or browser APIs
- Data fetching in Server Components — not in client components unless it's user-triggered
- `loading.tsx` for every route with async data
- `error.tsx` for every route that can fail
- Environment variables: `NEXT_PUBLIC_` only for values that must reach the client

## Styling (Tailwind)
- No inline `style={{}}` — use Tailwind classes
- Variant logic via `clsx` or `cn()` helper — not string concatenation
- No magic numbers in spacing — use Tailwind scale (4, 8, 12, 16...)
- Dark mode via `dark:` prefix, not JS toggle

## data-testid
- Add to ALL interactive elements and key containers per the feature spec
- Format: `[feature]-[component]-[element]` e.g. `login-form-submit`
- Never change a data-testid after E2E tests are written — breaking change

## Forms
- Use `react-hook-form` for non-trivial forms
- Validate with `zod` schema shared between client and server
- Show field-level errors inline, not in a toast
- Disable submit button while submitting; re-enable on error

## Performance
- Images via `next/image` always
- Fonts via `next/font` always
- Dynamic import (`next/dynamic`) for heavy components not needed on initial render
- No `useEffect` + `useState` for data that can be a Server Component

## Error handling
- API errors → show user-friendly message, log technical detail to console in dev
- Never expose stack traces or internal error messages to users
- Network errors and 5xx → same friendly message + retry option where it makes sense
- 4xx (validation) → show specific field errors from API response

## Anti-patterns to avoid
- `useEffect` for data fetching in client components when a Server Component would work
- Storing server state in `useState` — use React Query
- `any` type to silence TypeScript errors
- Prop drilling past 2 levels
- Array index as `key` in dynamic lists
