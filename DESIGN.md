# DESIGN.md — Auth Service

## What this service does

A FastAPI service that owns identity for other services: registration, login,
session refresh, logout, and a dependency other routes can use to require a
valid, non-revoked, logged-in user. Credentials are Postgres-backed; no other
service should ever see a raw password.

## What it promises the caller

- A registered password is never recoverable, only verifiable (argon2 hash).
- A valid session (access token) reliably identifies one user, until it
  expires or is explicitly revoked — whichever comes first.
- **Logout is immediate and absolute.** Once `/logout` returns, the access
  token that was live in that session is rejected on the very next request,
  even though it has not reached its `exp`. This is a real guarantee, not a
  client-side gesture.
- A refresh token can be used exactly once. Using an already-used one is
  treated as theft, not as an error to silently ignore — the entire session
  lineage is killed and the user has to log back in.

## The one failure this is designed against

**A token that should be dead still works.** Two ways that happens if you're
not careful: (1) you log out, but the access token — a self-contained,
signature-valid JWT — keeps authenticating you until it naturally expires;
(2) a refresh token leaks (XSS, log, shoulder-surf), and both the attacker
and the real user go on refreshing from it indefinitely, neither one ever
getting cut off. Every mechanism below exists to close one of these two
gaps — nothing here is defending against a third, unrelated threat.

## How that promise is kept

- **Revocation is checked on every request, not just at issue time.**
  `get_current_user` doesn't just verify the JWT signature and `exp` — it
  also checks the token's `jti` against a `denylist` table, in the same
  query that loads the user. A stateless JWT alone cannot make the logout
  promise above true; this check is what makes it true, at the cost of one
  DB round trip per authenticated request.
- **Refresh tokens are opaque, stored hashed, and rotate on every use.**
  The raw token is a random value the server never sees again after issuing
  it — only its SHA-256 lives in `refresh_tokens.token_hash`. Each refresh
  marks the old row revoked and issues a new one in the same `family_id`.
- **Reuse of a revoked refresh token kills the whole family.** If a
  token that's already marked `revoked_at` gets presented again, that is
  itself the signal something is wrong (a legitimate client never does
  this — it always throws its old token away after rotating). The response
  is to revoke every token sharing that `family_id`, not just deny the one
  request, and require a fresh login.
- **Logout revokes both halves of the session in one call.** The access
  token's `jti` goes on the denylist (pruned once past its own `exp` —
  no point keeping it after that); its `family_id` claim is used to revoke
  every refresh token in that family, with no extra lookup needed.

## Known tradeoffs / non-goals

- Denylist checks add one DB round trip to every authenticated request.
  Accepted at this scale; the documented scaling answer (not built here) is
  an in-memory or Redis cache in front of the same check.
- Denylist rows are pruned lazily (on login/logout), not by a background
  job. Acceptable for a single-instance dev service; a real deployment
  would run a periodic sweep instead.
- Tokens travel as httpOnly cookies, which means CSRF is a real concern for
  every mutating route (`/refresh`, `/logout`, anything behind
  `get_current_user` that isn't a plain GET). Mitigated with a double-submit
  `csrf_token` cookie + required `X-CSRF-Token` header, checked by a
  dedicated dependency — not solved by `SameSite` alone.
- No multi-device session listing, no "log out everywhere" UI, no rate
  limiting on login/refresh. Out of scope for this project; noted so it
  isn't mistaken for an oversight.
