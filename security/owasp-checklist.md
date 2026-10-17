\# OWASP Security Checklist



This checklist documents security controls and links to their implementation and tests.



| OWASP Category | Security Risk | Implementation Evidence | Test Evidence |

|---|---|---|---|

| A01 – Broken Access Control | Prevents customers from accessing administrator-only resources. | `src/middleware/rbac.js` | Test verifying that a customer token receives HTTP 403 when accessing an admin-only endpoint. |

| A02 – Cryptographic Failures | Protects passwords, refresh tokens, reset tokens, and authentication cookies. | Password hashing with bcrypt cost 12; refresh and reset tokens stored as SHA-256 hashes; secure cookies in production. | Tests verifying password hashing, token storage, and cookie security settings. |

| A03 – Injection | Validates input and prevents unsafe database queries. | Sequelize parameterised queries and express-validator rules. | Tests for input validation and database query handling. |

| A05 – Security Misconfiguration | Reduces exposure of internal errors and insecure configuration. | Production error handler hides stack traces; Helmet security headers; JWT secret length validation. | Tests for production error responses, security headers, and environment validation. |

| A07 – Identification and Authentication Failures | Protects authentication endpoints against repeated failures and token misuse. | Account lockout (T28), refresh-token reuse detection (T29), and rate limiting (T32). | Corresponding lockout, token reuse detection, and rate-limit tests. |

| A09 – Security Logging and Monitoring Failures | Prevents sensitive data from leaking through logs. | Logger redaction implementation. | Tests verifying that sensitive values are redacted from log output. |



\*\*Review requirement:\*\* Replace the suggested file paths with verified repository-relative links to the actual source files and tests. Confirm every link resolves to an existing file.

