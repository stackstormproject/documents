# Test Case Register: Admin Console and Admin API

Format per SENG 34213 s6.3.3. Paths are repository-relative (backend or stackstorm-frontend-admin). **Actual Output** and **Status** are filled during each execution; the last run is recorded below.

Last execution: 2026-10-27, local stack (PostgreSQL 16, Redis 7, backend branch `docs/157-admin-api-changelog`, admin branch `docs/153-admin-readme-changelog-docker`). Backend 584/584 Jest tests; admin 178/178 Vitest tests (99% lines); 11/11 Playwright tests (run twice).

## TC-SEC-122-01: Literal admin tokens are refused

| Field | Content |
|---|---|
| Test Case ID | TC-SEC-122-01 |
| Test Case Name | Literal admin tokens are refused |
| Related Requirement | NFR010 (A01/A07), #122 |
| Related Test File | backend: tests/unit/middleware/auth.test.js; tests/unit/socket/chatSocket.test.js; tests/unit/config/corsAllowList.test.js |
| Type | Unit |
| Priority | P1 |
| Preconditions | Any environment |
| Input | `Authorization: Bearer mock-admin-token` on /admin routes; socket with the same token |
| Expected Output | 401 over HTTP; socket role guest, not in admin_support_channel |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-SEC-132-01: Support agents cannot reach manager routes

| Field | Content |
|---|---|
| Test Case ID | TC-SEC-132-01 |
| Test Case Name | Support agents cannot reach manager routes |
| Related Requirement | NFR010 (A01), #132 |
| Related Test File | backend: tests/integration/admin/roleMatrix.test.js |
| Type | Integration |
| Priority | P1 |
| Preconditions | Agent and manager tokens |
| Input | Agent calls product, upload, order, stock, return, customer routes; then support sessions |
| Expected Output | 403 for each manager route; 200 for support sessions |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-SEC-139-01: A customer cannot become a chat agent

| Field | Content |
|---|---|
| Test Case ID | TC-SEC-139-01 |
| Test Case Name | A customer cannot become a chat agent |
| Related Requirement | FR032, NFR010 (A01), #139 |
| Related Test File | backend: tests/unit/socket/chatSocket.test.js |
| Type | Unit |
| Priority | P1 |
| Preconditions | Customer socket |
| Input | `chat:join` / `chat:message` with isAgent, senderRole, senderName |
| Expected Output | Not in agent channel; message stored as customer with name 'Customer' |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-SEC-139-02: A customer cannot open another customer's chat

| Field | Content |
|---|---|
| Test Case ID | TC-SEC-139-02 |
| Test Case Name | A customer cannot open another customer's chat |
| Related Requirement | FR032, NFR010 (A01), #139 |
| Related Test File | backend: tests/unit/socket/chatSocket.test.js |
| Type | Unit |
| Priority | P1 |
| Preconditions | Session owned by customer 7 |
| Input | Customer 42 joins it; posts to an unjoined room |
| Expected Output | Join refused; post refused; nothing stored |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-AUTH-143-01: No default or demo sign-in

| Field | Content |
|---|---|
| Test Case ID | TC-AUTH-143-01 |
| Test Case Name | No default or demo sign-in |
| Related Requirement | NFR010, #143 |
| Related Test File | admin: src/tests/shell/shell.test.jsx; e2e/auth.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Fresh browser |
| Input | Open / and /orders |
| Expected Output | Only the sign-in page; /login?next=%2Forders |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-AUTH-143-02: Wrong password message is neutral

| Field | Content |
|---|---|
| Test Case ID | TC-AUTH-143-02 |
| Test Case Name | Wrong password message is neutral |
| Related Requirement | NFR010 (A07), #143 |
| Related Test File | admin: src/tests/shell/shell.test.jsx; e2e/auth.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Seeded admin |
| Input | Correct email, wrong password |
| Expected Output | 'That email and password do not match a staff account.'; password cleared |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-AUTH-143-03: Session survives reload and ends on sign-out

| Field | Content |
|---|---|
| Test Case ID | TC-AUTH-143-03 |
| Test Case Name | Session survives reload and ends on sign-out |
| Related Requirement | NFR010, #143 |
| Related Test File | admin: e2e/auth.spec.js; src/tests/shell/shell.test.jsx |
| Type | E2E |
| Priority | P1 |
| Preconditions | Signed-in manager |
| Input | Reload; then sign out and open /orders |
| Expected Output | Still signed in after reload; after sign-out, /orders asks to sign in |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-AUTH-143-04: Support agent sees only Live support

| Field | Content |
|---|---|
| Test Case ID | TC-AUTH-143-04 |
| Test Case Name | Support agent sees only Live support |
| Related Requirement | #132, #143 |
| Related Test File | admin: src/tests/shell/shell.test.jsx; e2e/auth.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Seeded support@stackstorm.lk |
| Input | Sign in; open /orders |
| Expected Output | Lands on /support; menu has one link; /orders shows the role page |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-AUTH-142-01: Access token never stored

| Field | Content |
|---|---|
| Test Case ID | TC-AUTH-142-01 |
| Test Case Name | Access token never stored |
| Related Requirement | NFR010 (A07), #142 |
| Related Test File | admin: src/tests/lib/helpers.test.js |
| Type | Unit |
| Priority | P1 |
| Preconditions | None |
| Input | Set a token |
| Expected Output | localStorage holds no token; only the non-secret session hint |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-028-01: Dashboard counts paid orders only

| Field | Content |
|---|---|
| Test Case ID | TC-FR-028-01 |
| Test Case Name | Dashboard counts paid orders only |
| Related Requirement | FR028, FR029, #134, #144 |
| Related Test File | backend: tests/integration/admin/analytics.test.js; admin: src/tests/dashboard/dashboard.test.jsx |
| Type | Integration |
| Priority | P1 |
| Preconditions | Paid, unpaid and cancelled orders |
| Input | Open /dashboard; switch to 7 days |
| Expected Output | Revenue = paid orders; ?days=7; table view has one row per day |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-021-01: Find an order by number, email or name

| Field | Content |
|---|---|
| Test Case ID | TC-FR-021-01 |
| Test Case Name | Find an order by number, email or name |
| Related Requirement | FR021, #133, #145 |
| Related Test File | backend: tests/integration/admin/orderManagement.test.js; admin: src/tests/orders/orders.test.jsx; e2e/fulfilment.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Paid order |
| Input | Search its DFC number |
| Expected Output | The order is listed and opens |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-022-01: Shipping needs a tracking number

| Field | Content |
|---|---|
| Test Case ID | TC-FR-022-01 |
| Test Case Name | Shipping needs a tracking number |
| Related Requirement | FR022, #133, #145 |
| Related Test File | backend: orderManagement.test.js; admin: orders.test.jsx; e2e/fulfilment.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Processing home-delivery order |
| Input | Mark as shipped with empty, then 'RR123456789LK' |
| Expected Output | Asked for a number; then shipped with the number shown |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-022-02: Only legal transitions offered

| Field | Content |
|---|---|
| Test Case ID | TC-FR-022-02 |
| Test Case Name | Only legal transitions offered |
| Related Requirement | FR022, #133, #145 |
| Related Test File | backend: orderManagement.test.js; admin: orders.test.jsx |
| Type | Integration |
| Priority | P1 |
| Preconditions | Delivered order |
| Input | Open it; PATCH back to processing via API |
| Expected Output | No action buttons; API 409 |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-022-03: Cancelling a paid order restocks and refunds

| Field | Content |
|---|---|
| Test Case ID | TC-FR-022-03 |
| Test Case Name | Cancelling a paid order restocks and refunds |
| Related Requirement | FR022, FR023, #133, #145 |
| Related Test File | backend: orderManagement.test.js; e2e/fulfilment.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Paid order of 2 units, 10 in stock before |
| Input | Cancel order |
| Expected Output | Dialog states refund; toast 'Refunded LKR …'; stock back to 10 |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-023-01: Return approved, inspected and refunded once

| Field | Content |
|---|---|
| Test Case ID | TC-FR-023-01 |
| Test Case Name | Return approved, inspected and refunded once |
| Related Requirement | FR023, #137, #146 |
| Related Test File | backend: tests/integration/admin/returns.test.js; admin: src/tests/returns/returns.test.jsx; e2e/returns.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Delivered order, return requested |
| Input | Approve; record parcel as resellable |
| Expected Output | Item restocked; refund with key return-<id>; status completed; second decision 409 |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-023-02: Unacceptable return needs a reason and is not refunded

| Field | Content |
|---|---|
| Test Case ID | TC-FR-023-02 |
| Test Case Name | Unacceptable return needs a reason and is not refunded |
| Related Requirement | FR023, #137, #146 |
| Related Test File | backend: returns.test.js; admin: returns.test.jsx |
| Type | Integration |
| Priority | P1 |
| Preconditions | Approved return |
| Input | Record as not acceptable without, then with a note |
| Expected Output | Asked for a reason; then rejected, nothing refunded |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-023-03: Exchange only for the same product

| Field | Content |
|---|---|
| Test Case ID | TC-FR-023-03 |
| Test Case Name | Exchange only for the same product |
| Related Requirement | FR023, #155 |
| Related Test File | backend: tests/unit/services/returnService.test.js |
| Type | Unit |
| Priority | P1 |
| Preconditions | Returned item of product A |
| Input | Request exchange for product B; for the same variant; for another size of A |
| Expected Output | 400; 400; created |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-023-04: Exchange refused when the new size ran out

| Field | Content |
|---|---|
| Test Case ID | TC-FR-023-04 |
| Test Case Name | Exchange refused when the new size ran out |
| Related Requirement | FR023, #137, #146 |
| Related Test File | backend: returns.test.js; admin: returns.test.jsx |
| Type | Integration |
| Priority | P2 |
| Preconditions | Approved exchange, replacement stock 0 |
| Input | Record as resellable |
| Expected Output | 409 shown in the dialog; no stock moves |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-017-01: Create a product with sizes and opening stock

| Field | Content |
|---|---|
| Test Case ID | TC-FR-017-01 |
| Test Case Name | Create a product with sizes and opening stock |
| Related Requirement | FR017, #136, #147 |
| Related Test File | backend: tests/integration/admin/products.test.js; admin: src/tests/catalog/catalog.test.jsx; e2e/catalog-stock.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Manager |
| Input | New product with size M, opening stock 3 |
| Expected Output | SKU DFC-<id>-M-<COLOUR>; 3 available; history shows 'opening stock' |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-017-02: Archive hides a product; delete only if unsold

| Field | Content |
|---|---|
| Test Case ID | TC-FR-017-02 |
| Test Case Name | Archive hides a product; delete only if unsold |
| Related Requirement | FR017, #136, #147 |
| Related Test File | backend: products.test.js; admin: catalog.test.jsx; e2e/catalog-stock.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Sold and unsold products |
| Input | Archive; delete sold; delete unsold |
| Expected Output | Hidden from shop and checkout 409; 'Archive it instead'; unsold deleted |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-017-03: Ordered sizes keep size and colour

| Field | Content |
|---|---|
| Test Case ID | TC-FR-017-03 |
| Test Case Name | Ordered sizes keep size and colour |
| Related Requirement | FR017, #136, #147 |
| Related Test File | backend: products.test.js; admin: catalog.test.jsx |
| Type | Integration |
| Priority | P2 |
| Preconditions | Variant on an order |
| Input | Rename it; delete it; change extra price |
| Expected Output | 409; 409; saved |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-018-01: Stock adjustment cannot go below held units

| Field | Content |
|---|---|
| Test Case ID | TC-FR-018-01 |
| Test Case Name | Stock adjustment cannot go below held units |
| Related Requirement | FR017, FR018, ADR-004, #135, #148 |
| Related Test File | backend: tests/integration/admin/inventory.test.js; admin: src/tests/inventory/inventory.test.jsx |
| Type | Integration |
| Priority | P1 |
| Preconditions | 6 on hand, 2 held |
| Input | Remove 5; remove 4 |
| Expected Output | Refused (dialog and API 409); then 2 on hand and logged |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-018-02: Low-stock filter and history

| Field | Content |
|---|---|
| Test Case ID | TC-FR-018-02 |
| Test Case Name | Low-stock filter and history |
| Related Requirement | FR018, #135, #148 |
| Related Test File | admin: inventory.test.jsx; e2e/catalog-stock.spec.js |
| Type | Integration + E2E |
| Priority | P2 |
| Preconditions | Sizes under the threshold |
| Input | Tick Low stock only; restock with a note; open history |
| Expected Output | ?low=true; history shows 'restock: <note>' with the operator |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-024-01: Customer record and points adjustment

| Field | Content |
|---|---|
| Test Case ID | TC-FR-024-01 |
| Test Case Name | Customer record and points adjustment |
| Related Requirement | FR024, FR027, #138, #149 |
| Related Test File | backend: tests/integration/admin/customers.test.js; admin: src/tests/customers/customers.test.jsx |
| Type | Integration |
| Priority | P2 |
| Preconditions | Customer with 120 points |
| Input | Remove 500; add 50 without a reason; add 50 with a reason |
| Expected Output | Refused; reason required; balance 170, history shows the adjustment and admin |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-032-01: Agent takes a chat, replies and resolves

| Field | Content |
|---|---|
| Test Case ID | TC-FR-032-01 |
| Test Case Name | Agent takes a chat, replies and resolves |
| Related Requirement | FR032, FR033, #139, #150, #156 |
| Related Test File | admin: src/tests/support/support.test.jsx; e2e/support.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Guest shopper asks for an agent (real socket) |
| Input | Take this chat; reply; Mark resolved |
| Expected Output | Shopper sees the agent's name and reply; reply shown once; composer closed |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-NFR-008-01: No axe violations on any admin screen

| Field | Content |
|---|---|
| Test Case ID | TC-NFR-008-01 |
| Test Case Name | No axe violations on any admin screen |
| Related Requirement | NFR008, #151 |
| Related Test File | admin: src/tests/a11y/a11y.test.jsx |
| Type | Integration |
| Priority | P2 |
| Preconditions | Mocked API |
| Input | 12 screens, sign-in with errors, role page, 5 dialogs |
| Expected Output | 0 violations (contrast checked separately: lowest pair 4.83:1) |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-NFR-009-01: Admin console works at phone width

| Field | Content |
|---|---|
| Test Case ID | TC-NFR-009-01 |
| Test Case Name | Admin console works at phone width |
| Related Requirement | NFR009, #151 |
| Related Test File | admin: e2e/auth.spec.js (Pixel 5); manual 390 px check |
| Type | E2E + Manual |
| Priority | P2 |
| Preconditions | 390 px viewport |
| Input | Open every page; open and close the menu |
| Expected Output | No horizontal page scroll; closed menu not focusable; Escape closes it |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## Not covered

- Live Stripe refunds: automated runs use the mock provider; check one refund manually in Stripe test mode before release.
- nginx headers: no Docker daemon was available; run `docker run --rm stackstorm-admin nginx -t` and `curl -I` on staging.
