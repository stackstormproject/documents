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
| A01 / A07 | `mock-admin-token` granted super_admin outside production, including staging | backend `src/middleware/auth.js`, `src/socket/index.js` | **Fixed in #122**; see the admin console update below |
| A02 Cryptographic Failures | Card data | Card entry only in Stripe Elements iframes; the backend never receives card numbers (storefront `src/components/checkout/StripeCardForm.jsx`) | storefront `src/tests/checkout/checkout.test.jsx` |
| A04 Insecure Design | Payment trusted from the browser | Server retrieves the PaymentIntent and checks status, amount and order id before marking paid (backend `stripe.service.confirmOrderPayment`); webhook verified on the raw body (`src/app.js` `express.raw`) | backend `tests/unit/services/stripeService.test.js` |
| A05 Security Misconfiguration | Missing browser security headers | storefront `nginx.conf`, `security-headers.conf` (CSP, nosniff, frame and referrer policy on every location) | Manual: `curl -I` on staging; `nginx -t` in the image (see #120) |
| A07 Identification and Authentication Failures | Refresh-token reuse | backend `src/services/AuthService.js` (`refreshSession`, reuse revokes every session); cookie `HttpOnly`, `SameSite=Strict`, `Path=/api` | backend `tests/integration/auth/refresh.test.js`. The earlier row citing "T29" referred to code that did not exist before #105 |
| A07 Identification and Authentication Failures | Token theft through XSS | Access token in memory only; never written to web storage (storefront `src/lib/session.js`) | storefront `src/tests/lib/session.test.js` ("should never write the access token to storage") |
| A08 Software and Data Integrity | Lockfile integrity | `npm ci` in CI and the storefront Dockerfile | CI logs |

## Admin console update (2026-10-27, #122, #132-#157)

| OWASP Category | Risk addressed | Implementation evidence | Test evidence |
|---|---|---|---|
| A01 Broken Access Control | Literal tokens accepted as super_admin over HTTP and the socket | backend `src/middleware/auth.js`, `src/socket/index.js` (#122) | backend `tests/unit/middleware/auth.test.js`, `tests/unit/socket/chatSocket.test.js` |
| A01 Broken Access Control | Support agents could edit products and prices; every staff role had every admin route | backend `src/config/constants.js` (`MANAGER_ROLES`, `SUPPORT_ROLES`), `src/routes/admin.routes.js` (#132) | backend `tests/integration/admin/roleMatrix.test.js` |
| A01 Broken Access Control | A customer could join the agent channel by sending `isAgent: true`, read every chat, and post as the support desk; any client could join any session id | backend `src/socket/index.js` (agent status from the JWT role only; session ownership; joined rooms only) (#139) | backend `tests/unit/socket/chatSocket.test.js` (agent identity and session ownership) |
| A04 Insecure Design | Exchange for any product (a cheap item for a dear one) | backend `src/services/return.service.js` (same product, different variant) (#155) | backend `tests/unit/services/returnService.test.js` |
| A04 Insecure Design | Double refunds when a return is received twice | Row lock and status check per return; Stripe idempotency key `return-<id>` (#137) | backend `tests/integration/admin/returns.test.js` |
| A04 Insecure Design | Stock written below units held by live checkouts | `adjustStock` refuses `next < reserved_quantity` under a row lock (#135) | backend `tests/integration/admin/inventory.test.js` |
| A05 Security Misconfiguration | WebSocket accepted any origin with credentials (cross-site WebSocket hijacking) | Socket CORS reuses the HTTP allow-list `config/cors.js` (#122) | backend `tests/unit/config/corsAllowList.test.js` |
| A05 Security Misconfiguration | Admin pages framable, indexable, no CSP | admin `nginx.conf`, `security-headers.conf`: CSP with no third-party script, `frame-ancestors 'none'`, `X-Robots-Tag: noindex`, on every location (#153) | Manual: `nginx -t` in the image and `curl -I` on staging (no Docker daemon in the build environment) |
| A07 Identification and Authentication Failures | Hard-coded admin password auto-submitted by the login page; demo passwords in the bundle; default signed-in user | admin `src/state/AuthContext.jsx`, `src/pages/LoginPage.jsx` (no default user, no credential) (#143) | admin `src/tests/shell/shell.test.jsx`; `e2e/auth.spec.js` |
| A07 Identification and Authentication Failures | Admin token in localStorage | admin `src/lib/session.js` (memory only; refresh cookie restores) (#142) | admin `src/tests/lib/helpers.test.js` ("should never write the access token to storage") |
| A07 Identification and Authentication Failures | A shopper's refresh cookie accepted as a staff session | admin `src/api/auth.js` (`restoreSession` refuses a `customer` answer) (#142) | admin `src/tests/api/endpoints.test.js` |
| A01 Broken Access Control | Open redirect after staff sign-in | admin `src/lib/authErrors.js` (`safeNext`) (#143) | admin `src/tests/shell/shell.test.jsx` |
| A09 Security Logging and Monitoring Failures | Stock and points changes without an author | `inventory_logs.admin_id`, `loyalty_transactions.admin_id` shown in the console (#135, #138) | backend inventory and customer integration tests |
| A09 (open) | Order status changes, refunds, archive and delete are only in the application log | — | **Not done: issue #161** |
| A07 (open) | No staff password reset or two-factor sign-in for super admins | — | **Not done: issue #169** |

