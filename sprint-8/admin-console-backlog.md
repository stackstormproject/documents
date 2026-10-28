# Sprint 8: Admin Console Backlog

Issue descriptions for the admin console and the admin API, in the SENG 34213 s4.3 template. Paste each section into a GitHub issue, set **Phase = Development**, the epic label, the priority and the **Sprint 8** milestone.

> Numbers continue from the Sprint 7 backlog (#100-#131). #122 was raised there and is done here. If your board has moved on, renumber here and in the commit messages (`Closes #NN`) before opening the PRs.

## Epic

**As the owner of Dickwella Fashion Corner, I want my staff to run orders, returns, products, stock, customers and live chat from one secure console, so that the online shop can be operated without touching the database.**

## Summary and merge order

PRs are stacked: each branch starts from the one above it. Merge into `develop` top to bottom. Backend first: #122 removes the token the legacy admin app logs in with, so merge admin #141-#143 in the same release.

| # | Repo | Branch | Title | Epic | Priority | Est. |
|---|---|---|---|---|---|---|
| 122 | backend | `fix/122-remove-admin-backdoor-token` | Remove the mock-admin-token backdoor; restrict socket CORS | epic:security | P1-Critical | 3 h |
| 132 | backend | `fix/132-admin-role-matrix` | Restrict admin routes by role; return the admin profile | epic:auth | P1 | 3 h |
| 133 | backend | `feature/133-admin-order-management` | Admin order search, detail and status rules | epic:core-features | P1 | 8 h |
| 134 | backend | `fix/134-paid-revenue-analytics` | Count paid orders only; aggregate in SQL | epic:api | P1 | 3 h |
| 135 | backend | `feature/135-inventory-adjustments` | Stock list, logged adjustments, history | epic:core-features | P1 | 5 h |
| 136 | backend | `feature/136-admin-product-management` | Product and variant management with archiving | epic:core-features | P1 | 8 h |
| 137 | backend | `feature/137-admin-returns-workflow` | Return review, receiving, refunds and exchanges | epic:core-features | P1 | 8 h |
| 138 | backend | `feature/138-admin-customers-and-loyalty` | Customer records and audited loyalty adjustments | epic:core-features | P2 | 5 h |
| 139 | backend | `fix/139-support-agent-identity` | Chat agent identity from the verified token only | epic:security | P1-Critical | 3 h |
| 140 | backend | `chore/140-seed-support-agent` | Seed a support-agent account | epic:data-layer | P3 | 0.5 h |
| 155 | backend | `fix/155-exchange-same-product` | Exchanges limited to the same product | epic:security | P1 | 1.5 h |
| 156 | backend | `fix/156-support-message-senders` | Name agent messages; mark assistant messages | epic:api | P3 | 1 h |
| 157 | backend | `docs/157-admin-api-changelog` | Document the admin API and changelog | epic:documentation | P3 | 1 h |
| 141 | admin | `build/141-admin-quality-gates` | Team ESLint, Prettier and coverage gates | epic:ci-cd | P1 | 2 h |
| 142 | admin | `feature/142-admin-api-client` | API client and support socket matching the backend | epic:integration | P1 | 5 h |
| 143 | admin | `feature/143-admin-shell-auth-routing` | Real staff sign-in, URL routing, role-aware shell | epic:auth | P1-Critical | 8 h |
| 144 | admin | `feature/144-admin-dashboard` | Paid-revenue dashboard with trend and alerts | epic:ui-frontend | P2 | 5 h |
| 145 | admin | `feature/145-admin-order-management` | Order search, detail, guarded status changes | epic:ui-frontend | P1 | 8 h |
| 146 | admin | `feature/146-admin-returns-workflow` | Return review, inspection, refunds, exchanges | epic:ui-frontend | P1 | 6 h |
| 147 | admin | `feature/147-admin-catalog-management` | Product and size management with archiving | epic:ui-frontend | P1 | 8 h |
| 148 | admin | `feature/148-admin-inventory` | Stock list, adjustments and history | epic:ui-frontend | P1 | 5 h |
| 149 | admin | `feature/149-admin-customers-loyalty` | Customer records and points adjustments | epic:ui-frontend | P2 | 5 h |
| 150 | admin | `feature/150-admin-support-console` | Live support console | epic:ui-frontend | P2 | 6 h |
| 151 | admin | `test/151-admin-accessibility-responsive` | axe checks for every screen; responsive fixes | epic:testing | P2 | 3 h |
| 152 | admin | `test/152-admin-e2e-critical-flows` | Playwright E2E for critical admin flows | epic:testing | P1 | 5 h |
| 153 | admin | `docs/153-admin-readme-changelog-docker` | README, changelog, hardened Docker image | epic:documentation | P2 | 2 h |
| 154 | documents | `docs/154-admin-console-pack` | ADR-006, this backlog, test register, OWASP | epic:documentation | P2 | 3 h |

Total: **121 h** (backend 50 h, admin 68 h, documents 3 h). That is most of a sprint for the admin work alone at the Sprint 5 capacity of 160 h shared with other epics (documents/sprint-5/capacity.md). Suggested split: Sprint 8 = backend #122-#140, #155-#157 and admin #141-#148 (P1); Sprint 9 = #149-#154.

Raised but **not** done in this work: #158-#170 at the end of this file.

---

## Backend

### [DDP-#122] fix(security): remove the mock-admin-token backdoor and restrict socket CORS

**Labels:** epic:security, P1-Critical, fix · **Estimate:** 3 h · **Dependencies:** none (merge with admin #141-#143)

#### User Story
As the shop owner, I want only real staff accounts to reach the admin API, so that nobody can change orders or prices by sending a known string.

#### Background / Context
- SRS Reference: NFR010 (OWASP Top 10)
- SDS Reference: Security design, auth middleware
- ADR Reference: ADR-003, ADR-006

`middleware/auth.js` accepted `mock-admin-token` and `jwt-admin-token-xyz` as `super_admin` when `NODE_ENV !== 'production'`. The socket server did the same, skipped the issuer check, defaulted unknown ids to admin 1 and accepted any origin with credentials (cross-site WebSocket hijacking).

#### Acceptance Criteria
**AC1: Literal tokens refused**
Given a request with `Authorization: Bearer mock-admin-token`
When it calls any `/api/v1/admin` route
Then 401 is returned in every environment

**AC2: Socket treats them as guests**
Given a socket connecting with `auth.token = 'mock-admin-token'`
When it connects
Then its role is `guest` and it is not in `admin_support_channel`

**AC3: Socket CORS**
Given a page on an origin not in the HTTP allow-list
When it opens a socket with credentials
Then the handshake is refused

#### Technical Notes
- Socket uses the same `verifyAccessToken` and `toRequestUser` as HTTP; CORS reuses `config/cors.js` (`isOriginAllowed`).

#### Test Requirements
- [x] Unit: socket guest fallback, CORS allow-list
- [x] Unit: literal tokens rejected by the auth middleware (`tests/unit/middleware/auth.test.js`)

#### Definition of Done
- [x] All AC tests pass · [x] lint and format clean · [ ] PR reviewed · [x] CHANGELOG updated (#157)

---

### [DDP-#132] fix(rbac): restrict admin routes by role and return the admin profile

**Labels:** epic:auth, P1-High, fix · **Estimate:** 3 h · **Dependencies:** #122

#### User Story
As a store manager, I want support agents to reach only the support queue, so that a chat agent cannot change prices or refund orders.

#### Background / Context
- SRS Reference: NFR010; stakeholder roles (SRS s2.3)
- Every admin route used one `ADMIN_ROLES` list, so `support_agent` could edit products. There was no `GET /auth/admin/me`.

#### Acceptance Criteria
**AC1: Managers**
Given a token with role `admin` or `super_admin`
When it calls catalogue, upload, order, stock, return or customer admin routes
Then they are allowed

**AC2: Agents**
Given a token with role `support_agent`
When it calls any of those routes
Then 403 is returned, and `GET /admin/support/sessions` returns 200

**AC3: Profile**
Given a staff token
When `GET /auth/admin/me` is called
Then `{ admin: { id, name, email, role } }` is returned

#### Technical Notes
- `MANAGER_ROLES`, `SUPPORT_ROLES` in `config/constants.js`; `rbac(roles)` per router.

#### Test Requirements
- [x] Integration: `tests/integration/admin/roleMatrix.test.js` (table of routes x roles)

#### Definition of Done
- [x] All AC tests pass · [x] lint and format clean · [ ] PR reviewed

---

### [DDP-#133] feat(orders): admin order search, detail and status rules

**Labels:** epic:core-features, P1-High, feature · **Estimate:** 8 h · **Dependencies:** #132

#### User Story
As a store manager, I want to find an order and move it to shipped, delivered or cancelled with the right side effects, so that customers get their parcel or their money back.

#### Background / Context
- SRS Reference: FR021, FR022
- ADR Reference: ADR-004 (reservations), ADR-006

#### Acceptance Criteria
**AC1: Search**
Given orders in the database
When `GET /admin/orders?search=DFC-2026-0011` (or an email or name) and `status=` are used
Then matching orders are returned newest first with pagination

**AC2: Allowed transitions**
Given an order
When `GET /admin/orders/:id` is called
Then `allowed_statuses` lists only legal next states: payment_pending → cancelled; processing → shipped | cancelled; shipped → delivered; store pickup processing → delivered

**AC3: Tracking required**
Given a processing home-delivery order
When it is set to shipped without `tracking_number`
Then 400 is returned and nothing changes

**AC4: Paid cancellation**
Given a paid order
When it is cancelled
Then stock is returned, the payment is refunded and the response includes `refund`

**AC5: Unpaid cancellation**
Given a payment_pending order
When it is cancelled
Then its stock hold is released and no refund is attempted

**AC6: Delivery date**
Given a shipped order
When it is set to delivered
Then `delivery_date` is set (starts the 30-day return window)

#### Technical Notes
- Row lock on the order; refunds outside the transaction; `stripe.service.refundOrderPayment`.

#### Test Requirements
- [x] Integration: each AC, illegal transitions return 409

#### Definition of Done
- [x] All AC tests pass · [x] lint and format clean · [ ] PR reviewed

---

### [DDP-#134] fix(analytics): count paid orders only and aggregate in SQL

**Labels:** epic:api, P1-High, fix · **Estimate:** 3 h · **Dependencies:** #132

#### User Story
As the owner, I want revenue figures to include only orders that were paid, so that I do not plan stock on sales that never happened.

#### Background / Context
- SRS Reference: FR028, FR029
- `getOverview` summed every order in memory, including unpaid and cancelled ones.

#### Acceptance Criteria
**AC1: Paid only**
Given paid, unpaid and cancelled orders
When `GET /admin/analytics` is called
Then revenue, paid orders and average order value count confirmed, processing, shipped and delivered orders only

**AC2: Trend window**
Given `GET /admin/analytics/sales-trend?days=7`
When it is called
Then one entry per day for 7 days is returned, zero-filled, with totals

**AC3: Alerts**
Given low stock and open returns
When the overview is read
Then `low_stock_count`, `low_stock_threshold`, `open_returns` and `support_queue_count` are present

#### Test Requirements
- [x] Integration: paid-only totals, zero-filled trend

#### Definition of Done
- [x] All AC tests pass · [x] lint and format clean · [ ] PR reviewed

---

### [DDP-#135] feat(inventory): stock list, logged adjustments and history

**Labels:** epic:core-features, P1-High, feature · **Estimate:** 5 h · **Dependencies:** #132

#### User Story
As a store manager, I want to correct stock with a reason and see who changed it, so that the shop never sells what is not on the shelf.

#### Background / Context
- SRS Reference: FR017, FR018
- ADR Reference: ADR-004 (held units)

#### Acceptance Criteria
**AC1: List**
Given variants
When `GET /admin/inventory?search=&low=true` is called
Then each row has on hand, reserved, available and `is_low` against the threshold

**AC2: Adjust**
Given a variant with 6 on hand and 2 held
When `POST /admin/inventory/:id/adjust` with `{ delta: -5, reason: 'damaged' }`
Then 409 is returned (below held) and nothing changes; with `delta: -4` it succeeds and is logged with the admin id

**AC3: Reasons**
Given reason `other` without a note
When adjusting
Then 400 is returned

**AC4: History**
Given adjustments
When `GET /admin/inventory/:id/logs` is called
Then entries are newest first with the admin's name

#### Test Requirements
- [x] Integration: `tests/integration/admin/inventory.test.js` (9 cases)

#### Definition of Done
- [x] All AC tests pass · [x] lint and format clean · [ ] PR reviewed

---

### [DDP-#136] feat(catalog): product and variant management with archiving

**Labels:** epic:core-features, P1-High, feature · **Estimate:** 8 h · **Dependencies:** #132, #135

#### User Story
As a store manager, I want to add products with their sizes and retire old ones without losing order history, so that the catalogue stays current and accurate.

#### Background / Context
- SRS Reference: FR003, FR017
- Products had no active flag; deleting one broke order lines. `POST /products` let any staff role write.

#### Acceptance Criteria
**AC1: Create**
Given a name, price, category and at least one size
When `POST /admin/products` is called
Then the product and its variants are created, SKUs are `DFC-<id>-<SIZE>-<COLOUR>`, and opening stock is logged

**AC2: Archive**
Given an active product
When `PUT /admin/products/:id { is_active: false }`
Then it disappears from the shop, its detail page returns 404, and checkout refuses it with 409

**AC3: Delete only if unsold**
Given a product that was ordered or is held
When `DELETE /admin/products/:id`
Then 409 "Archive it instead" is returned

**AC4: Variants in use**
Given a variant that was ordered
When its size or colour is changed or it is deleted
Then 409 is returned; changing its extra price is allowed; the last variant cannot be removed

#### Test Requirements
- [x] Integration: create, archive visibility, delete rules, variant rules
- [x] Unit: product filter excludes inactive products

#### Definition of Done
- [x] All AC tests pass · [x] lint and format clean · [ ] PR reviewed

---

### [DDP-#137] feat(returns): return review, receiving, refunds and exchanges

**Labels:** epic:core-features, P1-High, feature · **Estimate:** 8 h · **Dependencies:** #133

#### User Story
As a store manager, I want to approve a return, inspect the parcel and refund or send the new size, so that customers are treated fairly and stock stays right.

#### Background / Context
- SRS Reference: FR023; UC-005
- No admin return routes existed.

#### Acceptance Criteria
**AC1: Approve or reject**
Given a requested return
When it is approved (optional note) or rejected (note required)
Then the status changes once; a second decision returns 409

**AC2: Receive, acceptable return**
Given an approved return
When `POST /admin/returns/:id/receive { acceptable: true }`
Then the item is restocked, the refund amount is refunded with idempotency key `return-<id>`, and the status is completed

**AC3: Receive, acceptable exchange**
Given an approved exchange
When received as acceptable
Then the returned item is restocked and the replacement deducted; if the replacement is out of stock, 409 and nothing changes

**AC4: Not acceptable**
Given an approved return
When received as not acceptable without a note
Then 400; with a note it is rejected and nothing is refunded

#### Test Requirements
- [x] Integration: `tests/integration/admin/returns.test.js` (8 cases); unit: refund idempotency key

#### Definition of Done
- [x] All AC tests pass · [x] lint and format clean · [ ] PR reviewed

---

### [DDP-#138] feat(crm): customer records and audited loyalty adjustments

**Labels:** epic:core-features, P2-Medium, feature · **Estimate:** 5 h · **Dependencies:** #132

#### User Story
As a store manager, I want to look up a customer's orders and adjust their points with a reason, so that I can resolve complaints with a goodwill credit.

#### Background / Context
- SRS Reference: FR024, FR027
- The legacy app called `/loyalty/*` routes that do not exist. `redeemPoints` had a race (no row lock).

#### Acceptance Criteria
**AC1: Search and sort**
Given customers
When `GET /admin/customers?search=&tier=&sort=spend` is called
Then each row has tier, points, paid order count, paid spend and last order

**AC2: Record**
Given a customer
When `GET /admin/customers/:id` is called
Then recent orders, addresses and the last 20 points entries (with admin name) are returned

**AC3: Adjust**
Given 120 points
When `POST /admin/customers/:id/loyalty-adjustments { points: -500, note }`
Then 409; without a note 400; `{ points: 50, note }` succeeds and is logged as `adjusted`

#### Test Requirements
- [x] Integration: `tests/integration/admin/customers.test.js` (9 cases); unit: redeem row lock

#### Definition of Done
- [x] All AC tests pass · [x] lint and format clean · [ ] PR reviewed

---

### [DDP-#139] fix(support): take chat agent identity from the verified token only

**Labels:** epic:security, P1-Critical, fix · **Estimate:** 3 h · **Dependencies:** #122

#### User Story
As a customer, I want my chat to be visible only to me and to real support staff, so that my order details stay private.

#### Background / Context
- SRS Reference: FR032, NFR010 (A01)
- A client sending `isAgent: true` (or `senderRole`, `role`, `sender`) joined `admin_support_channel` and saw every conversation; any client could join another customer's session by id.

#### Acceptance Criteria
**AC1: No promotion by payload**
Given a customer socket
When it sends `chat:join` and `chat:message` with `isAgent: true`
Then it is not in the agent channel and its messages are stored as the customer's

**AC2: Session ownership**
Given a session owned by customer 7
When customer 42 joins it
Then the join is refused

**AC3: Joined rooms only**
Given a customer who has not joined session 303
When they post to it
Then the message is refused and nothing is stored

**AC4: Agent name**
Given a support agent
When they send a message
Then it is signed with their name from the admins table

#### Test Requirements
- [x] Unit: `tests/unit/socket/chatSocket.test.js` (5 new cases)

#### Definition of Done
- [x] All AC tests pass · [x] lint and format clean · [ ] PR reviewed

---

### [DDP-#140] chore(seed): add a support-agent account

**Labels:** epic:data-layer, P3-Low, chore · **Estimate:** 0.5 h · **Dependencies:** none

#### User Story
As a developer, I want a seeded support agent, so that I can try and test the support-only view.

#### Acceptance Criteria
**AC1:** Given a fresh `npm run db:seed`, When it finishes, Then `support@stackstorm.lk` exists with role `support_agent` and the `SEED_ADMIN_PASSWORD` password.

#### Definition of Done
- [x] Seed runs · [ ] PR reviewed

---

### [DDP-#155] fix(returns): limit exchanges to another variant of the same product

**Labels:** epic:security, P1-High, fix · **Estimate:** 1.5 h · **Dependencies:** #137

#### User Story
As the shop owner, I want an exchange to be another size or colour of the same item, so that nobody swaps a cheap tee for a silk sari.

#### Background / Context
- SRS Reference: FR023. Found while building admin #146.

#### Acceptance Criteria
**AC1:** Given a returned item of product A, When an exchange for a variant of product B is requested, Then 400 is returned.
**AC2:** Given the same variant as the returned one, When requested, Then 400 is returned.
**AC3:** Given another size of product A, When requested, Then the exchange is created.

#### Test Requirements
- [x] Unit: 3 cases in `returnService.test.js`; the #137 integration test now uses another size

#### Definition of Done
- [x] All AC tests pass · [ ] PR reviewed

---

### [DDP-#156] fix(support): name agent messages and keep the assistant's apart

**Labels:** epic:api, P3-Low, fix · **Estimate:** 1 h · **Dependencies:** #139

#### User Story
As a support agent, I want to see who wrote each message, so that I can tell the virtual assistant's answers from a colleague's.

#### Acceptance Criteria
**AC1:** Given a session with customer, bot and agent messages, When `GET /admin/support/sessions` is called, Then senders are `customer`, `bot` ("Virtual assistant") and `agent` (the agent's name, or "Support desk" without an agent id).

#### Test Requirements
- [x] Integration: `tests/integration/admin/supportSessions.test.js`

#### Definition of Done
- [x] AC test passes · [ ] PR reviewed

---

### [DDP-#157] docs(api): document the admin API, seeded staff accounts and changelog

**Labels:** epic:documentation, P3-Low, docs · **Estimate:** 1 h · **Dependencies:** #156

#### User Story
As a new developer, I want the README to list the admin routes and seeded accounts, so that I can start the console without reading the code.

#### Acceptance Criteria
**AC1:** Given the README, When followed on a fresh machine, Then migrate, seed and both staff accounts are covered and the admin route table lists roles.
**AC2:** Given the CHANGELOG, When read, Then #122, #132-#140, #155 and #156 are listed under [Unreleased].

---

## Admin console

### [DDP-#141] build(admin): replace oxlint with the team ESLint, Prettier and coverage gates

**Labels:** epic:ci-cd, P1-High, build · **Estimate:** 2 h · **Dependencies:** none

#### User Story
As a reviewer, I want the admin app checked by the same gates as the other repos, so that CI catches what the coding standard forbids.

#### Background / Context
- Standard: documents/standards/coding-standards.md (Airbnb). The app used oxlint and committed `coverage/`. A real lint run: 211 errors in 48 files.

#### Acceptance Criteria
**AC1:** Given `npm run lint`, When run, Then ESLint (Airbnb, hooks, jsx-a11y, prettier) runs with `--max-warnings 0`.
**AC2:** Given `npm run test:coverage`, When coverage of new code is under 80%, Then it fails.
**AC3:** Given `npm run verify`, When run, Then lint, format check, coverage and build run in that order.
**AC4:** Given the repo, When cloned, Then no coverage report or `.env` is tracked and `.env.example` documents each variable.

#### Technical Notes
- Legacy files that fail lint are listed in `.eslintignore` and excluded from coverage until replaced (empty after #143).

#### Definition of Done
- [x] Gates run in CI's shared workflow · [ ] PR reviewed

---

### [DDP-#142] feat(admin): API client and support socket that match the backend contract

**Labels:** epic:integration, P1-High, feature · **Estimate:** 5 h · **Dependencies:** #141; backend #132-#139

#### User Story
As a developer, I want one client that speaks the real admin API, so that screens do not invent data or call missing routes.

#### Acceptance Criteria
**AC1: Envelope and errors**
Given any API call
When the backend answers in any of its three error shapes
Then the caller receives `ApiError { status, code, message, details }`

**AC2: Session**
Given an expired access token
When two requests fail with `TOKEN_EXPIRED`
Then one `POST /auth/refresh` runs and both retry; a shopper session is not accepted as staff

**AC3: Contract**
Given each admin module
When called
Then it hits the route, verb and body the backend serves (contract tests)

**AC4: Socket**
Given the support socket
When joining or replying
Then no role flags are sent, and every reconnect presents the current in-memory token

#### Test Requirements
- [x] `src/tests/lib/http.test.js`, `src/tests/api/endpoints.test.js`, `src/tests/api/supportSocket.test.js`

#### Definition of Done
- [x] 100% lines on new modules · [ ] PR reviewed

---

### [DDP-#143] feat(admin): real staff sign-in, URL routing and a role-aware shell

**Labels:** epic:auth, P1-Critical, feature · **Estimate:** 8 h · **Dependencies:** #142; backend #122

#### User Story
As a staff member, I want to sign in with my own account and land on the screens my role allows, so that the console is safe to leave open on the shop computer.

#### Background / Context
- SRS Reference: NFR010; ADR-003, ADR-006
- The app defaulted to a signed-in super admin with a literal token, auto-submitted a hard-coded password, and shipped demo passwords.

#### Acceptance Criteria
**AC1: Sign-in only**
Given no session
When any console URL is opened
Then the sign-in page is shown and the URL is kept as `?next=` (same-site paths only)

**AC2: Errors**
Given wrong credentials, a locked account or too many attempts
When signing in
Then a plain message is shown and the password field is cleared

**AC3: Restore**
Given a signed-in manager
When the page is reloaded
Then the session is restored from the refresh cookie without signing in again

**AC4: Roles**
Given a support agent
When signed in
Then they land on /support, the menu shows only Live support, and other URLs show "not available for your role"

**AC5: Session end**
Given a refresh that fails mid-work
When the next request runs
Then the operator is signed out with "Your session ended"

**AC6: No secrets**
Given the production bundle
When searched
Then it contains no password or token literal

#### Test Requirements
- [x] `src/tests/shell/shell.test.jsx` (21 cases), `src/tests/ui/ui.test.jsx`; E2E `e2e/auth.spec.js`

#### Definition of Done
- [x] All AC tests pass · [x] legacy facade removed · [ ] PR reviewed

---

### [DDP-#144] feat(dashboard): paid-revenue dashboard with sales trend and alerts

**Labels:** epic:ui-frontend, P2-Medium, feature · **Estimate:** 5 h · **Dependencies:** #143; backend #134

#### User Story
As the owner, I want to see paid revenue and what needs attention when I open the console, so that I can act on low stock, returns and waiting customers first.

#### Acceptance Criteria
**AC1:** Given the overview, When the dashboard loads, Then revenue, paid orders, average order and active customers are shown, from paid orders only.
**AC2:** Given alerts, When shown, Then awaiting payment, open returns, low stock and support queue each link to the filtered list behind the number.
**AC3:** Given the trend, When 7, 30 or 90 days is chosen, Then the chart reloads, the range is in the URL, and the same numbers are available as a table.
**AC4:** Given a support agent, When opening /dashboard, Then the role page is shown and no analytics call is made.

#### Test Requirements
- [x] `src/tests/dashboard/dashboard.test.jsx` (8)

---

### [DDP-#145] feat(orders): order search, detail and guarded status changes

**Labels:** epic:ui-frontend, P1-High, feature · **Estimate:** 8 h · **Dependencies:** #143; backend #133

#### User Story
As a store manager, I want to find an order, ship it with a tracking number, and cancel it with a clear refund message, so that customers receive accurate updates.

#### Acceptance Criteria
**AC1: List**
Given orders
When searching by number, email or name and filtering by status
Then the list updates, keeps filters and page in the URL, and marks guest orders

**AC2: Only legal actions**
Given an order
When its page opens
Then only the actions in `allowed_statuses` are offered

**AC3: Ship**
Given a processing home-delivery order
When "Mark as shipped" is confirmed without a tracking number
Then the dialog asks for one; with one, the order shows shipped and the number

**AC4: Cancel**
Given a paid order
When "Cancel order" is opened
Then the dialog states the restock and the refund amount; after confirming, a toast reports the refund (or an error toast if the refund failed)

**AC5: Store pickup**
Given a pickup order
When delivered
Then the action reads "Mark as collected" and tracking is optional

#### Test Requirements
- [x] `src/tests/orders/orders.test.jsx` (16); E2E `e2e/fulfilment.spec.js`

---

### [DDP-#146] feat(returns): return review, parcel inspection, refunds and exchanges

**Labels:** epic:ui-frontend, P1-High, feature · **Estimate:** 6 h · **Dependencies:** #143; backend #137, #155

#### User Story
As a store manager, I want to decide on return requests and record the parcel's condition, so that refunds and exchanges happen only for acceptable items.

#### Acceptance Criteria
**AC1:** Given requests, When the list opens, Then new requests come first and status filters are in the URL.
**AC2:** Given a request made more than 30 days after delivery, When opened, Then a warning is shown.
**AC3:** Given a requested return, When rejected without a reason, Then the dialog asks for one.
**AC4:** Given an approved return, When "Record returned parcel" is opened, Then the dialog states the refund amount (or the size to send) before saving; "Not acceptable" needs a description.
**AC5:** Given an exchange whose size ran out, When saved, Then the server's message is shown in the dialog and nothing changes.

#### Test Requirements
- [x] `src/tests/returns/returns.test.jsx` (14); E2E `e2e/returns.spec.js`

---

### [DDP-#147] feat(catalog): product and size management with archiving

**Labels:** epic:ui-frontend, P1-High, feature · **Estimate:** 8 h · **Dependencies:** #143; backend #136

#### User Story
As a store manager, I want to add a product with its sizes and archive it when the season ends, so that the shop shows what we actually sell.

#### Acceptance Criteria
**AC1:** Given the new-product form, When saved with missing name, category, price or size, Then each problem is shown and focus moves to the first.
**AC2:** Given valid details and sizes, When saved, Then the product opens with its SKUs and opening stock.
**AC3:** Given a product, When archived or restored, Then the shop visibility changes after confirming.
**AC4:** Given a sold product, When deleting, Then the server's "archive it instead" is shown.
**AC5:** Given an ordered size, When edited, Then only the extra price can change, and it cannot be removed; the last size cannot be removed.

#### Test Requirements
- [x] `src/tests/catalog/catalog.test.jsx` (21); E2E `e2e/catalog-stock.spec.js`

---

### [DDP-#148] feat(inventory): stock list, logged adjustments and stock history

**Labels:** epic:ui-frontend, P1-High, feature · **Estimate:** 5 h · **Dependencies:** #143; backend #135

#### User Story
As a store manager, I want to restock or write off units with a reason, so that stock matches the shelf and every change can be traced.

#### Acceptance Criteria
**AC1:** Given stock, When "Low stock only" is ticked, Then only sizes at or under the threshold are shown and the filter is in the URL.
**AC2:** Given a size with 6 on hand and 2 held, When removing 5, Then the dialog refuses ("only 4 can be removed").
**AC3:** Given reason "other", When saving without a note, Then the dialog asks for one.
**AC4:** Given an adjustment, When saved, Then the row updates in place and the history lists it with the operator's name.

#### Test Requirements
- [x] `src/tests/inventory/inventory.test.jsx` (9); E2E `e2e/catalog-stock.spec.js`

---

### [DDP-#149] feat(crm): customer records and audited loyalty adjustments

**Labels:** epic:ui-frontend, P2-Medium, feature · **Estimate:** 5 h · **Dependencies:** #143; backend #138

#### User Story
As a store manager, I want to open a customer's record and give goodwill points with a reason, so that complaints are settled consistently.

#### Acceptance Criteria
**AC1:** Given customers, When filtered by tier and sorted by spend, Then the URL keeps both.
**AC2:** Given a record, When opened, Then paid spend, orders (linked), addresses, and points history (with who adjusted) are shown.
**AC3:** Given 120 points, When removing 500, Then the dialog refuses; a reason is always required.

#### Test Requirements
- [x] `src/tests/customers/customers.test.jsx` (9)

---

### [DDP-#150] feat(support): live support console on verified agent identity

**Labels:** epic:ui-frontend, P2-Medium, feature · **Estimate:** 6 h · **Dependencies:** #143; backend #139, #156

#### User Story
As a support agent, I want to see who is waiting, take a chat and reply in real time, so that shoppers get help before they leave.

#### Acceptance Criteria
**AC1: Queue**
Given waiting chats
When /support opens
Then Waiting, Active and Resolved tabs show counts, and new chats appear without reloading

**AC2: Take and reply**
Given a waiting chat
When "Take this chat" is used
Then the customer is told the agent's name and the composer is enabled; Enter sends, and each message appears once

**AC3: Typing and resolve**
Given an active chat
When the customer types, Then "is typing…" shows; When "Mark resolved" is used, Then the composer closes for both sides

**AC4: Failures**
Given a refused join or a send timeout
When it happens
Then the message is shown in the conversation and the draft is kept

#### Test Requirements
- [x] `src/tests/support/support.test.jsx` (10); E2E `e2e/support.spec.js` (real shopper socket)

---

### [DDP-#151] test(a11y): axe checks for every screen and responsive fixes

**Labels:** epic:testing, P2-Medium, test · **Estimate:** 3 h · **Dependencies:** #150

#### User Story
As a staff member using a keyboard, screen reader or phone, I want every screen to work, so that the console meets NFR008.

#### Acceptance Criteria
**AC1:** Given each of the 12 screens and 5 dialogs, When axe-core runs, Then there are no violations.
**AC2:** Given a 390 px wide screen, When any page opens, Then there is no horizontal page scroll.
**AC3:** Given the phone menu is closed, When tabbing, Then its links are not focusable; Escape closes it.
**AC4:** Given the palette, When checked, Then every text and badge pair is at least 4.5:1.

---

### [DDP-#152] test(e2e): Playwright suite for the critical admin flows

**Labels:** epic:testing, P1-High, test · **Estimate:** 5 h · **Dependencies:** #151

#### User Story
As the team, I want the top admin flows tested against the real API, so that a regression in any layer is caught before a release.

#### Acceptance Criteria
**AC1:** Given `E2E_ADMIN_PASSWORD`, When `npm run test:e2e` runs against a local or staging stack, Then sign-in and roles, ship and deliver, cancel with refund, product and stock, return with refund, and live chat pass.
**AC2:** Given each test, When it runs, Then it creates its own data through the API.
**AC3:** Given no password, When the suite runs, Then it is skipped, not failed, and no password is committed.

---

### [DDP-#153] docs(admin): README, changelog and hardened Docker image

**Labels:** epic:documentation, P2-Medium, docs · **Estimate:** 2 h · **Dependencies:** #152

#### Acceptance Criteria
**AC1:** Given the README, When followed on a fresh machine, Then the console runs and the seeded accounts are named (s9.3 sections, badges).
**AC2:** Given the image, When built, Then `npm ci` is used and nginx sends CSP, frame-ancestors none and noindex on every location.

---

### [DDP-#154] docs: admin console pack

**Labels:** epic:documentation, P2-Medium, docs · **Estimate:** 3 h · **Dependencies:** none

#### Acceptance Criteria
**AC1:** ADR-006, this backlog, `testing/test-register-admin.md` and the OWASP update are in the documents repo.

---

## Raised, not done (create these issues)

| # | Repo | Title | Why it matters | Priority |
|---|---|---|---|---|
| 158 | backend | Low-stock email alerts and a configurable threshold | FR018 asks for notifications and configurable thresholds; the threshold is a constant (5) and only the dashboard shows it | P2 |
| 159 | backend + admin | Live order list updates | FR021 asks for real-time updates; the order list refreshes on navigation only | P3 |
| 160 | backend + admin | Staff account management | No screen or route creates, disables or changes staff accounts; today it needs the seed or SQL | P1 |
| 161 | backend | Audit log for admin actions | Stock and points changes are logged; order status changes, refunds, archive and delete are only in the app log | P2 |
| 162 | backend + admin | Bulk product upload (CSV, images ZIP) | FR019 | P3 |
| 163 | backend + admin | Inventory analytics (stock value, turnover, dead stock) | FR020 | P3 |
| 164 | backend + admin | Sales reports with exports and period comparison | FR028 asks for weekly/monthly/yearly reports and CSV/Excel/PDF export | P3 |
| 165 | backend | Return labels through the courier API | FR023 | P3 |
| 166 | backend + admin | Wishlist, reviews and chat history on the customer record | FR024 | P3 |
| 167 | backend + admin | Chat transfer, agent availability and attachments | FR032 | P3 |
| 168 | backend + admin | Upload product photos instead of pasting links | The product form takes an https link; the S3 upload route exists but is not wired to the form | P2 |
| 169 | backend | Staff password reset and two-factor sign-in for super admins | Admins cannot recover a forgotten password; super admin has full control | P2 |
| 170 | admin | Order filters by date range and amount | FR021 lists date and amount filters; search and status are done | P3 |
