\# ADR-003: Authentication Design



\## Status

Proposed — pending team and supervisor review.



\## Context

StackStorm requires secure authentication and session management for customers and administrators. The design must support token renewal, session revocation, and account lockout after repeated failed login attempts.



\## Decision



\### Token Lifetimes

Access tokens and refresh tokens must have defined expiration periods configured according to the agreed authentication implementation.



\### Cookie Settings

Authentication cookies must use appropriate security settings, including `HttpOnly`, `Secure` in HTTPS environments, and an appropriate `SameSite` policy.



\### Refresh Token Rotation and Revocation

Refresh tokens must be stored as hashes in the database. When a refresh token is used to renew a session, the previous token must be invalidated and replaced. Tokens must support expiration and revocation.



\### Refresh Token Reuse Detection

The authentication implementation must detect reuse of an invalidated refresh token and apply the agreed session-revocation response.



\### Account Lockout

Customer and administrator accounts must track failed login attempts using `failed\_login\_attempts` and `locked\_until`. The authentication implementation must enforce the agreed lockout policy.



\### Password Hashing

Passwords must be hashed using bcrypt. Plain-text passwords must not be stored.



\### Refresh Token Ownership

The `refresh\_tokens` table must support ownership by either a customer or an administrator. A database CHECK constraint must ensure that exactly one of `customer\_id` and `admin\_id` is populated.



\## Alternatives

\- Store refresh tokens as plain text: rejected because it increases the impact of database exposure.

\- Allow refresh tokens to belong to both or neither owner: rejected because token ownership must be unambiguous.

\- Omit account lockout: rejected because it does not provide the agreed protection against repeated failed login attempts.



\## Consequences

\- The database requires a refresh token table and account lockout columns.

\- Authentication logic must manage token expiration, rotation, revocation, and reuse detection.

\- Password verification must use bcrypt.

\- Tests must cover token ownership and account lockout behaviour.



\## SDS Impact

Update the Software Design Specification ER diagram to include `refresh\_tokens`, its relationships with `customers` and `admins`, and the `failed\_login\_attempts` and `locked\_until` columns in both owner tables.



\## Review

Pending teammate approval and supervisor review.

