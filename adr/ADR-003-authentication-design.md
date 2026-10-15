# ADR-003: Authentication Design

| Field | Value |
|---|---|
| Date | 2026-10-15 |
| Status | Proposed — pending supervisor review and confirmation against the authentication implementation |
| Issue | [#14](https://github.com/stackstormproject/documents/issues/14) |
| SDS reference | Security Design and Entity Relationship diagram |
| Related backend work | [#30](https://github.com/stackstormproject/backend/issues/30), [#32](https://github.com/stackstormproject/backend/issues/32), [#31](https://github.com/stackstormproject/backend/pull/31), and [#33](https://github.com/stackstormproject/backend/pull/33) |

## Context

The application has customer and administrator accounts and needs authentication that limits credential exposure, supports session renewal and revocation, and reduces the impact of stolen or reused credentials. The backend authentication utilities and database schema are being developed in separate changes. Their documented defaults include a one-hour access token, a seven-day refresh token, bcrypt work factor 12, and login lockout after five failed attempts for 15 minutes.

The documents repository currently contains no SDS PDF or editable ER diagram. The accompanying [authentication ER diagram source](../sds/authentication-er-diagram.md) records the authentication-related schema from backend issue #32 pending incorporation into the authoritative SDS.

## Decision

Adopt the following design. Values described as proposed must be confirmed during implementation review before this ADR is accepted.

### Token lifetimes and password storage

- Issue signed JWT access tokens using HS256 and the `stackstorm-api` issuer. Default access-token lifetime is one hour; deployments may configure it with `JWT_EXPIRES_IN`.
- Issue opaque refresh tokens with a seven-day default lifetime, configurable through `REFRESH_TOKEN_EXPIRES_IN`. Persist only their SHA-256 hashes, never their raw values.
- Hash passwords with bcrypt using work factor 12 by default, configurable through `BCRYPT_WORK_FACTOR`. Production configuration must not use a lower work factor.

### Cookie settings

Store the refresh token in a cookie with `HttpOnly`, `Secure` over HTTPS, and `SameSite=Lax`. Scope its path to the refresh/logout endpoints, omit the `Domain` attribute, and set its expiry to the refresh-token lifetime. Do not place refresh tokens in browser local storage. The exact cookie path and deployment-specific HTTPS behavior must be confirmed in the backend implementation.

### Refresh-token rotation and revocation

- Rotate the refresh token on successful use: mark the presented token as used and persist only the hash of its successor.
- Revoke refresh tokens on logout and when the owning account's credentials require invalidation.
- Perform consumption/rotation atomically so parallel requests cannot both create independent successors.

### Refresh-token reuse detection

Detect use of an already-consumed refresh token. The backend defines a 10-second `REFRESH_REUSE_GRACE_SECONDS` setting; use this only to accommodate concurrent/retried refresh requests. Reuse outside that window should revoke the owner's active refresh tokens and require a new login.

The grace-window behavior and revocation scope above are design decisions to verify against the authentication flow; the database fields alone do not establish the complete implementation.

### Account lockout

After five consecutive failed login attempts, lock the customer or administrator account for 15 minutes. Track the attempt count and lock expiry in `failed_login_attempts` and `locked_until`. Reset the counter after a successful login and when the lockout has expired.

### Refresh-token ownership

Each `refresh_tokens` row belongs to exactly one account: either a customer or an administrator, never both and never neither. Enforce this in the database with a check constraint, in addition to foreign keys. Store token hash, expiry, use, revocation, and creation timestamps so refresh credentials can be expired, rotated, and revoked.

## Alternatives

### Long-lived access tokens without refresh tokens

Rejected because a stolen access token would remain useful for longer and could not be individually revoked through persisted session state.

### Store raw refresh tokens

Rejected because a database disclosure would immediately expose usable refresh credentials. Store one-way hashes instead.

### One polymorphic owner field for both account types

Rejected because a generic owner identifier would not provide database-enforced foreign keys to both account tables. Two nullable foreign keys plus an exclusive-owner check provide referential integrity and enforce exactly one owner.

### Store browser tokens in local storage

Rejected for refresh credentials because JavaScript-accessible storage increases exposure to script injection. An `HttpOnly` cookie reduces that exposure; `SameSite` and CSRF protections must still be applied consistently.

## Consequences

**Positive:**

- Short-lived access tokens limit the window of use if an access token is stolen.
- Hashed, rotated refresh tokens support revocation and reduce the value of a database leak.
- Bcrypt and bounded lockouts make password guessing and offline password attacks more costly.
- Database constraints prevent ownerless or ambiguously owned refresh-token records.

**Negative / operational considerations:**

- Refresh-token rotation, reuse detection, and revocation require atomic database operations and tests for concurrent requests.
- Cookie-based authentication requires correct HTTPS, CORS, `SameSite`, and CSRF configuration across environments.
- A 15-minute lockout can be abused to deny service to a targeted account; monitor lockout behavior and provide a safe recovery path.
- Token lifetimes and lockout settings must be configurable without exposing secrets or logging raw credentials.

## SDS Impact

Update the SDS ER diagram to add `refresh_tokens` with nullable `customer_id` and `admin_id` foreign keys, a constraint requiring exactly one owner, a unique token hash, expiry and lifecycle timestamps, and the relationships to `customers` and `admins`. Add `failed_login_attempts` and `locked_until` to both account tables.

An ER diagram source is included at [sds/authentication-er-diagram.md](../sds/authentication-er-diagram.md). The repository does not currently contain the full SDS document, so the source must be incorporated into the authoritative SDS when it is available. Supervisor review and confirmation against completed authentication work are still required before this ADR can be marked Accepted.