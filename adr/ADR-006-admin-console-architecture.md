# ADR-006: Admin Console Architecture

- **Status:** Proposed (for supervisor review within one sprint, SENG 34213 s1.3)
- **Date:** 2026-10-27
- **Tickets:** stackstorm-frontend-admin #141-#153; backend #122, #132-#140, #155-#157
- **Related:** SRS FR017, FR018, FR021-FR024, FR027-FR029, FR032, FR033, NFR008, NFR010; ADR-003 (authentication); ADR-004 (stock reservation); ADR-005 (customer storefront)

## Context

An audit of the admin app and the admin API on 2026-10-27, run against the real backend on PostgreSQL 16 and Redis, found that the console could not run the shop and was itself a security hole:

- **Anyone was an administrator.** The backend accepted the literal tokens `mock-admin-token` and `jwt-admin-token-xyz` as `super_admin` whenever `NODE_ENV` was not `production` (staging included). The admin app's auth context started signed in with that token, the login page auto-submitted `admin@stackstorm.lk` / `AdminPassword123!`, and a `DEMO_ADMIN_ACCOUNTS` list with passwords shipped in the bundle.
- **Customers could become support agents.** The chat socket treated any client that sent `isAgent: true` (or `senderRole`, `role`, `sender`) as an agent and put it in `admin_support_channel`, where every conversation is broadcast.
- **Nothing on screen was real when the API failed.** Every page started from `FALLBACK_*` arrays and kept them on error, so staff could "ship" orders that did not exist.
- **The console called routes that do not exist:** `PATCH /admin/returns/:id/approve|reject`, `/loyalty/accounts|adjust|audit`, `POST /inventory/adjust`, `POST /products`. The backend had no admin order, return, stock, customer or product management at all.
- **Analytics counted unpaid and cancelled orders as revenue.**
- **One component switched views with a `currentTab` flag**, so nothing had a URL. Tokens were in `localStorage`.
- Lint was oxlint, not the team's Airbnb config, and a coverage report was committed.

## Decision

1. **Server-enforced roles, mirrored in the UI.** Managers (`super_admin`, `admin`) run catalogue, stock, orders, returns and customers; `support_agent` only reaches support (`MANAGER_ROLES`, `SUPPORT_ROLES` in `config/constants.js`, #132). The console builds its menu from the same matrix (`src/lib/roles.js`) and shows a "not available for your role" page instead of a screen of 403s.
2. **Same authentication model as the storefront (ADR-003, ADR-005).** `POST /auth/admin/login`, access token in memory only, refresh token in the httpOnly cookie scoped to `/api`, single-flight refresh with rotation. A refresh that returns a shopper (`customer`) is not accepted as a staff session. No default user and no credential in the client.
3. **Socket identity from the JWT only (#139).** Agent status, sender role and sender name come from the verified token and the `admins` table. Customers may only join their own session and post into rooms they joined.
4. **The server owns every business rule; the UI asks and re-reads.**
   - Order transitions: `allowed_statuses` is returned with the order and the console shows only those buttons; shipping needs a tracking number; cancelling a paid order restocks and refunds (#133).
   - Returns: requested → approved → received (completed or rejected); refunds use an idempotency key per return; exchanges are limited to another size of the same product (#137, #155).
   - Stock: adjustments are signed deltas with a fixed reason list, logged with the operator, and cannot go below units held by ADR-004 reservations (#135).
   - Products: variants that were ordered or are held keep their size and colour; a product that was ever ordered can be archived but not deleted (#136).
   - Loyalty: manual adjustments need a reason and never take the balance below zero (#138).
   - Revenue: paid statuses only, aggregated in SQL (#134).
5. **Every screen and list filter has a URL** (React Router 7 data router, filters in the query string), so a filtered list or a single order can be bookmarked and shared between staff.
6. **One HTTP client, one error shape** (`src/lib/http.js`, shared design with the storefront): the backend's three error envelopes become `ApiError { status, code, message, details }`; server refusals (409s) are shown in the dialog that caused them.
7. **Loading, error-with-retry and empty states on every page; no fallback data.**
8. **Live support is lazy-loaded** so socket.io is only downloaded by staff who open it.
9. **Quality gates:** Airbnb ESLint with zero warnings, Prettier, Vitest with an 80% threshold, axe-core on every screen and dialog, Playwright for the critical flows, `npm run verify` = CI.

## Alternatives considered

| Option | Why not |
|---|---|
| Keep the legacy app and patch endpoints | Default sign-in, fallbacks and the tab switcher are structural; patching would leave the backdoor's dependants in place |
| Hide forbidden menu items only, without server checks | The API is the security boundary; the menu is convenience (OWASP A01) |
| Let the client compute allowed order transitions | Rules would exist twice and drift; `allowed_statuses` keeps one source of truth |
| Optimistic UI for status changes and refunds | A refund or restock that silently failed would look done; the console re-reads after every change |
| Optimistic chat messages matched by text and time (old approach) | Two identical replies merged; the server echo with a message id is exact |
| A separate admin auth service / session cookies | ADR-003 already gives rotation and reuse detection; one model for both apps |
| Redux Toolkit or React Query | `useResource` plus two contexts (auth, toasts) cover the needs for a 4-person team |

## Consequences

- The backend #122 fix breaks the legacy admin app; merge backend #122-#140 and admin #141-#143 together (merge order in `sprint-8/admin-console-backlog.md`).
- Locally, the storefront and console on `localhost` share the refresh cookie (signing in to one signs out the other). Deployed on separate hosts they do not.
- There is no staff account management screen yet: staff accounts are created by seed or SQL (raised as #160).
- Tier upgrades and points accrual remain automatic and unimplemented (#129); the console only adjusts points.
- The E2E order flows rely on the backend's mock payment provider, so staging must run without a live Stripe key or the flows must be skipped.
- The SDS must be updated with the role matrix, the order and return state machines above, and the socket identity rule.
