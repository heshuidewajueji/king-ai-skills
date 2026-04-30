# Feature Spec — [Feature Name]

**Priority:** P0 / P1 / P2  
**Status:** pending / in_progress / done  
**Complexity:** S / M / L  
**Depends on:** [other feature names, or "none"]

---

## Description

[What this feature does and why it exists. 2–4 sentences.]

---

## Acceptance Criteria

> Each criterion must be testable. Written as: "Given [context], when [action], then [outcome]."

- [ ] Given an unauthenticated user, when they visit `/login`, then the login form is displayed
- [ ] Given valid credentials, when the user submits the form, then they are redirected to `/dashboard`
- [ ] Given invalid credentials, when the user submits the form, then an error message is shown
- [ ] Given a network error, when the user submits the form, then a user-friendly error is shown and the form is not cleared

---

## UI Elements & data-testid Map

> All interactive elements and key containers must have a data-testid.
> Naming convention: `[feature]-[component]-[element]`

| Element | data-testid | Notes |
|---------|-------------|-------|
| Email input | `login-form-email` | |
| Password input | `login-form-password` | |
| Submit button | `login-form-submit` | Disabled while submitting |
| Error message | `login-form-error` | Only visible on error |
| Loading spinner | `login-form-loading` | Visible during API call |

---

## API / Data Contract

> Define request/response shapes for non-trivial endpoints.

```
POST /api/auth/login
Request:  { email: string, password: string }
Response: { user: { id, email, name }, token: string }
Errors:   401 { error: "Invalid credentials" }
          429 { error: "Too many attempts" }
          500 { error: "Internal server error" }
```

---

## Edge Cases & Error States

- Empty form submission → show validation errors inline, do not call API
- Rate limiting (429) → show "Too many attempts, try again in X minutes"
- Session already active → redirect to dashboard without showing login form
- Password forgotten flow → link to `/forgot-password` (separate feature)

---

## Out of Scope

> Explicitly list what is NOT part of this feature to prevent scope creep.

- Social login (OAuth) — separate feature
- Two-factor authentication — separate feature
- Remember me / persistent sessions — separate feature
