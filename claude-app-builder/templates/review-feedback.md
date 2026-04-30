# Review Feedback — [Feature Name]

**Date:** [date]  
**Reviewer:** Agent 2 (Code Reviewer)

---

## Status: APPROVED | CHANGES_REQUIRED

---

## Blocking Issues

> Must be fixed before tests can run. Reference file:line for each issue.

- [ ] `src/components/LoginForm.tsx:34` — Missing `data-testid="login-form-error"` on error element
- [ ] `src/app/api/auth/login/route.ts:12` — No rate limiting check, spec requires 429 handling

---

## Non-blocking Suggestions

> Improvements that don't block progress but should be considered.

- `src/components/LoginForm.tsx:28` — Consider adding `autocomplete="email"` for better UX
- Consider extracting form validation logic into a custom hook for reusability

---

## Spec Compliance

- [ ] All acceptance criteria implemented
- [ ] All `data-testid` attributes present and correctly named
- [ ] All edge cases handled
- [ ] All error states handled  
- [ ] No scope creep (no features added beyond spec)

---

## Security

- [ ] No secrets in client-side code
- [ ] Input sanitization in place
- [ ] Auth tokens handled securely (httpOnly cookies or equivalent)

---

## Type Safety

- [ ] No `any` types
- [ ] API response types validated
- [ ] All async operations properly typed
