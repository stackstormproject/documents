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


## Customer path update (2026-10-27, #100-#120)

Verified against the code on the branches listed in `sprint-7/customer-storefront-backlog.md`.

| OWASP Category | Risk addressed | Implementation evidence | Test evidence |
|---|---|---|---|
| A01 Broken Access Control | Anyone could read any registered customer's order; any logged-in user could mark any order paid | backend `src/services/order.service.js` (`canReadOrder`), `src/services/stripe.service.js` (`loadOwnedOrder`); `POST /orders/:id/finalize` removed | backend `tests/integration/orders/orderAccess.test.js`, `tests/integration/checkout/payment.test.js` |
| A01 Broken Access Control | Open redirect after sign-in | storefront `src/lib/authErrors.js` (`safeNext`) | storefront `src/tests/auth/auth.test.jsx` |
| A01 / A07 (open) | `mock-admin-token` grants super_admin outside production, including staging | backend `src/middleware/auth.js`, `src/socket/index.js`; admin `src/context/AdminAuthContext.jsx` | **Not fixed: issue #122** |
| A02 Cryptographic Failures | Card data | Card entry only in Stripe Elements iframes; the backend never receives card numbers (storefront `src/components/checkout/StripeCardForm.jsx`) | storefront `src/tests/checkout/checkout.test.jsx` |
| A04 Insecure Design | Payment trusted from the browser | Server retrieves the PaymentIntent and checks status, amount and order id before marking paid (backend `stripe.service.confirmOrderPayment`); webhook verified on the raw body (`src/app.js` `express.raw`) | backend `tests/unit/services/stripeService.test.js` |
| A05 Security Misconfiguration | Missing browser security headers | storefront `nginx.conf`, `security-headers.conf` (CSP, nosniff, frame and referrer policy on every location) | Manual: `curl -I` on staging; `nginx -t` in the image (see #120) |
| A07 Identification and Authentication Failures | Refresh-token reuse | backend `src/services/AuthService.js` (`refreshSession`, reuse revokes every session); cookie `HttpOnly`, `SameSite=Strict`, `Path=/api` | backend `tests/integration/auth/refresh.test.js`. The earlier row citing "T29" referred to code that did not exist before #105 |
| A07 Identification and Authentication Failures | Token theft through XSS | Access token in memory only; never written to web storage (storefront `src/lib/session.js`) | storefront `src/tests/lib/session.test.js` ("should never write the access token to storage") |
| A08 Software and Data Integrity | Lockfile integrity | `npm ci` in CI and the storefront Dockerfile | CI logs |
