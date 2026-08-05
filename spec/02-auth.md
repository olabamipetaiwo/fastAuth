# Project 2: Auth Service Done Properly

> Part of the **FastAPI Path**. Read `00-README.md` first. Every project ships three things: a one-page DESIGN note before code, the implementation, and a NOTES.md answering the questions at the end.

**Effort:** about a week
**Prerequisite:** Project 1, or equivalent comfort with routes and a database

## Read This First

Almost everyone copies auth from a tutorial and never understands it, which is why so much production auth is quietly broken. Build it once, deliberately, and you will understand FastAPI's dependency injection system better than any other exercise can teach you, because the guard that protects a route is exactly what dependencies were designed for.

The trap is treating this as "add login." The content is in the parts the tutorials skip: token expiry, refresh, and the fact that a stateless token cannot be un-issued.

## What You'll Learn

- Password hashing, and why you hash rather than encrypt
- FastAPI dependency injection as an authorization mechanism (`get_current_user`)
- JWT issuance, expiry, and validation
- Refresh tokens and rotation
- The logout problem: revoking a token that is still cryptographically valid

## Core Task

Build a service with registration, login, protected routes that require a valid token, token refresh, and logout. A logged-out token must stop working even if it has not expired.

## Build Components

1. **Registration.** Accept credentials, hash the password with bcrypt or argon2 (never store the password, never "encrypt" it, and be able to say why hashing and encryption are different), store the user.
2. **Login.** Verify the password against the hash, issue a short-lived access token and a longer-lived refresh token.
3. **The auth dependency.** A FastAPI dependency that extracts the token, validates signature and expiry, loads the user, and injects it. Protected routes declare this dependency and get the user for free. An invalid or expired token returns 401.
4. **Refresh.** Exchange a valid refresh token for a new access token. Rotate the refresh token on use so a stolen one has a limited window.
5. **Logout and revocation.** Here is the real work. An access token is valid until it expires, so "logout" cannot just delete it client-side. Maintain a denylist (or switch to short access tokens plus a revocable refresh token) so a logged-out token is rejected before its expiry.

## The Verifier

A script that: registers and logs in; hits a protected route with the token and asserts 200; hits it with no token, a garbage token, and an expired token and asserts 401 each; refreshes and asserts the old refresh token no longer works (rotation); logs out and asserts the still-unexpired access token is now rejected.

## The Uncomfortable Part

A JWT is stateless by design, which is the feature everyone wants and the reason logout is hard. If your access tokens live 20 minutes and you have no denylist, then "logout" is a lie: the token works for up to 20 more minutes no matter what your UI says. You will discover there is no clean fix, only tradeoffs: very short access tokens (more refresh traffic), a denylist (stateful, which partly defeats the point of JWT), or session tokens instead of JWTs. Pick one and defend it. There is no answer that is free.

## NOTES.md

1. Why hash instead of encrypt a password? What can an attacker who steals your database do in each case?
2. Walk through exactly what your `get_current_user` dependency does on every request. What is the cost per request, and does it hit the database every time?
3. State your logout mechanism and its cost. How long, worst case, can a logged-out token still be used?
4. Your refresh tokens rotate. What happens if an attacker and the real user both hold the same refresh token and both try to use it? (This is the token-reuse-detection problem. Describe what you do or explicitly what you don't.)

## Resources

- The FastAPI security tutorial, the full OAuth2-with-JWT section, read completely
- The FastAPI docs on dependencies, especially dependencies with yield and sub-dependencies
- OWASP guidance on password storage and on JWT handling
- Any clear writeup on refresh token rotation and reuse detection


Postgres + SQLAlchemy (async)
Postgres + raw SQL