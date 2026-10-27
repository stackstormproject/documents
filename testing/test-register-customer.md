# Test Case Register: Customer Storefront

Format per SENG 34213 s6.3.3. Paths are repository-relative (backend or stackstorm-frontend-customer). **Actual Output** and **Status** are filled during each execution; the last run is recorded below.

Last execution: 2026-10-27, local stack (PostgreSQL 16, Redis 7, backend branch `chore/107-seed-variants-for-all-products`, storefront branch `docs/120-readme-changelog-docker`). Backend 505/505 Jest tests, storefront 242/242 Vitest tests, 13/13 Playwright tests.

## TC-FR-001-01: Search returns matching products

| Field | Content |
|---|---|
| Test Case ID | TC-FR-001-01 |
| Test Case Name | Search returns matching products |
| Related Requirement | FR001 |
| Related Test File | stackstorm-frontend-customer: src/tests/catalog/catalog.test.jsx; e2e/catalog.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Seeded catalog |
| Input | Search "batik" from the header |
| Expected Output | /shop?search=batik; title Results for "batik"; matching products listed |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-002-01: Filters are applied by the server and kept in the URL

| Field | Content |
|---|---|
| Test Case ID | TC-FR-002-01 |
| Test Case Name | Filters are applied by the server and kept in the URL |
| Related Requirement | FR002 |
| Related Test File | src/tests/catalog/catalog.test.jsx; e2e/catalog.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Catalog open |
| Input | Choose size M and LKR 3,000 to 6,000; reload |
| Expected Output | URL has size=M&price=3000-6000; API called with min/max price; selection survives reload |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-003-01: Stock labels per variant

| Field | Content |
|---|---|
| Test Case ID | TC-FR-003-01 |
| Test Case Name | Stock labels per variant |
| Related Requirement | FR003 |
| Related Test File | src/tests/lib/helpers.test.js; src/tests/product/product.test.jsx |
| Type | Unit + Integration |
| Priority | P1 |
| Preconditions | Variants with 0, 2 and 12 available |
| Input | Open product |
| Expected Output | Out of stock (size disabled), Only 2 left, In stock |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-004-01: Product gallery and details

| Field | Content |
|---|---|
| Test Case ID | TC-FR-004-01 |
| Test Case Name | Product gallery and details |
| Related Requirement | FR004 |
| Related Test File | src/tests/product/product.test.jsx |
| Type | Integration |
| Priority | P2 |
| Preconditions | Product with 2 images |
| Input | Click thumbnail 2 |
| Expected Output | Main image changes; material and care shown |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-005-01: Wishlist for customers only

| Field | Content |
|---|---|
| Test Case ID | TC-FR-005-01 |
| Test Case Name | Wishlist for customers only |
| Related Requirement | FR005 |
| Related Test File | backend tests/integration/customerIdentity.test.js; src/tests/catalog/catalog.test.jsx |
| Type | Integration |
| Priority | P2 |
| Preconditions | Guest, then customer |
| Input | Toggle heart |
| Expected Output | Guest sent to sign in (API 401 AUTH_REQUIRED); customer item saved and listed |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-006-01: Server cart with product details

| Field | Content |
|---|---|
| Test Case ID | TC-FR-006-01 |
| Test Case Name | Server cart with product details |
| Related Requirement | FR006 |
| Related Test File | backend tests/integration/cart/cart.test.js; src/tests/cart/cart.test.jsx |
| Type | Integration |
| Priority | P1 |
| Preconditions | Guest session token |
| Input | Add variant x2, GET /cart |
| Expected Output | Line has product name, unit_price, line_total, available_stock |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-006-02: Customer cart and guest-cart merge

| Field | Content |
|---|---|
| Test Case ID | TC-FR-006-02 |
| Test Case Name | Customer cart and guest-cart merge |
| Related Requirement | FR006, NFR006 |
| Related Test File | backend tests/integration/customerIdentity.test.js; e2e/account.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Guest bag with 1 item, existing account |
| Input | Sign in |
| Expected Output | Bag count shows the guest item in the customer cart |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-006-03: Checkout blocked when a line exceeds stock

| Field | Content |
|---|---|
| Test Case ID | TC-FR-006-03 |
| Test Case Name | Checkout blocked when a line exceeds stock |
| Related Requirement | FR006 |
| Related Test File | src/tests/cart/cart.test.jsx |
| Type | Integration |
| Priority | P1 |
| Preconditions | Line quantity 3, available 1 |
| Input | Open bag |
| Expected Output | "Only 1 left" shown; Check out disabled |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-008-01: Only verified purchasers may review

| Field | Content |
|---|---|
| Test Case ID | TC-FR-008-01 |
| Test Case Name | Only verified purchasers may review |
| Related Requirement | FR008, BR |
| Related Test File | backend tests/unit/services/reviewService.test.js; src/tests/product/product.test.jsx |
| Type | Unit + Integration |
| Priority | P2 |
| Preconditions | Customer without a paid order |
| Input | Post 5-star review |
| Expected Output | HTTP 403; UI explains only buyers can review |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-009-01: Order history

| Field | Content |
|---|---|
| Test Case ID | TC-FR-009-01 |
| Test Case Name | Order history |
| Related Requirement | FR009 |
| Related Test File | backend tests/integration/orders/orderAccess.test.js; src/tests/account/account.test.jsx |
| Type | Integration |
| Priority | P1 |
| Preconditions | Customer with 1 order |
| Input | Open /account/orders |
| Expected Output | Invoice number, status, items count, total |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-010-01: Order tracking timeline

| Field | Content |
|---|---|
| Test Case ID | TC-FR-010-01 |
| Test Case Name | Order tracking timeline |
| Related Requirement | FR010 |
| Related Test File | src/tests/account/account.test.jsx; e2e/account.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Paid order |
| Input | Open order detail |
| Expected Output | Current step marked; tracking number only if entered by the shop |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-013-01: Checkout prices from the server

| Field | Content |
|---|---|
| Test Case ID | TC-FR-013-01 |
| Test Case Name | Checkout prices from the server |
| Related Requirement | FR013, FR016 |
| Related Test File | backend tests/integration/checkout/initiate.test.js |
| Type | Integration |
| Priority | P1 |
| Preconditions | LKR 4,200 product, 5 in stock |
| Input | Customer checks out 2, standard delivery |
| Expected Output | 201; subtotal 8,400, shipping 450, total 8,850; reservation linked to order |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-013-02: Payment verified by the server and stock deducted

| Field | Content |
|---|---|
| Test Case ID | TC-FR-013-02 |
| Test Case Name | Payment verified by the server and stock deducted |
| Related Requirement | FR013, NFR011 |
| Related Test File | backend tests/integration/checkout/payment.test.js |
| Type | Integration |
| Priority | P1 |
| Preconditions | payment_pending order, 5 in stock, 2 ordered |
| Input | create-intent then confirm |
| Expected Output | Order processing; reservation CONFIRMED; stock 3; reserved 0 |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-013-03: Forged or foreign payment rejected

| Field | Content |
|---|---|
| Test Case ID | TC-FR-013-03 |
| Test Case Name | Forged or foreign payment rejected |
| Related Requirement | FR013, OWASP A01 |
| Related Test File | backend tests/integration/checkout/payment.test.js |
| Type | Integration (negative) |
| Priority | P1 |
| Preconditions | Order of customer A |
| Input | Confirm without token / with forged intent id |
| Expected Output | 404 / 400; order stays payment_pending |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-013-04: Card decline shown, retry possible

| Field | Content |
|---|---|
| Test Case ID | TC-FR-013-04 |
| Test Case Name | Card decline shown, retry possible |
| Related Requirement | FR013 |
| Related Test File | src/tests/checkout/checkout.test.jsx |
| Type | Integration (negative) |
| Priority | P1 |
| Preconditions | Stripe provider |
| Input | Stripe returns a decline |
| Expected Output | Message "Your card was declined."; server confirm not called |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-014-01: Guest checkout end to end

| Field | Content |
|---|---|
| Test Case ID | TC-FR-014-01 |
| Test Case Name | Guest checkout end to end |
| Related Requirement | FR014, UC-003 |
| Related Test File | e2e/guest-checkout.spec.js; src/tests/checkout/checkout.test.jsx |
| Type | E2E |
| Priority | P1 |
| Preconditions | Running backend, seeded stock |
| Input | Browse, add, address, pay |
| Expected Output | Confirmation page; status Being prepared; bag 0 items |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-016-01: Free delivery threshold and collection

| Field | Content |
|---|---|
| Test Case ID | TC-FR-016-01 |
| Test Case Name | Free delivery threshold and collection |
| Related Requirement | FR016, BR |
| Related Test File | backend tests/integration/checkout/initiate.test.js; src/tests/checkout/checkout.test.jsx |
| Type | Integration |
| Priority | P1 |
| Preconditions | Subtotal 10,000 / collection chosen |
| Input | Checkout |
| Expected Output | Shipping 0; summary says Free |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-016-02: Invalid postal code refused

| Field | Content |
|---|---|
| Test Case ID | TC-FR-016-02 |
| Test Case Name | Invalid postal code refused |
| Related Requirement | FR016 |
| Related Test File | e2e/guest-checkout.spec.js |
| Type | E2E (negative) |
| Priority | P2 |
| Preconditions | Bag with item |
| Input | Postal code 812 |
| Expected Output | Field error; no order or hold created |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-ADR4-01: Hold expiry releases stock

| Field | Content |
|---|---|
| Test Case ID | TC-ADR4-01 |
| Test Case Name | Hold expiry releases stock |
| Related Requirement | ADR-004 |
| Related Test File | src/tests/checkout/checkout.test.jsx; backend tests/integration/inventory/expirySweep.test.js |
| Type | Integration |
| Priority | P1 |
| Preconditions | Hold created |
| Input | Countdown reaches 0 |
| Expected Output | Release called; shopper told the hold ended |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-023-01: Return rules enforced

| Field | Content |
|---|---|
| Test Case ID | TC-FR-023-01 |
| Test Case Name | Return rules enforced |
| Related Requirement | FR023, BR |
| Related Test File | backend tests/unit/services/returnService.test.js; src/tests/account/account.test.jsx |
| Type | Unit + Integration |
| Priority | P1 |
| Preconditions | Delivered 30 / 31 days ago, final sale, shipped |
| Input | Request return |
| Expected Output | Allowed at 30 days; refused at 31, final sale, not delivered |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-023-02: Exchange for another in-stock size

| Field | Content |
|---|---|
| Test Case ID | TC-FR-023-02 |
| Test Case Name | Exchange for another in-stock size |
| Related Requirement | FR023, UC-005 |
| Related Test File | src/tests/account/account.test.jsx |
| Type | Integration |
| Priority | P2 |
| Preconditions | Delivered order, sizes M and L in stock |
| Input | Choose exchange, size M |
| Expected Output | Request sent with exchange_variant_id; sold-out sizes not offered |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-027-01: Loyalty tier and points shown

| Field | Content |
|---|---|
| Test Case ID | TC-FR-027-01 |
| Test Case Name | Loyalty tier and points shown |
| Related Requirement | FR027 |
| Related Test File | src/tests/account/account.test.jsx |
| Type | Integration |
| Priority | P3 |
| Preconditions | Gold customer 1,250 points |
| Input | Open /account |
| Expected Output | Gold member; 1,250 points to spend |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-035-01: Registration and sign-in

| Field | Content |
|---|---|
| Test Case ID | TC-FR-035-01 |
| Test Case Name | Registration and sign-in |
| Related Requirement | FR035, UC-001 |
| Related Test File | src/tests/auth/auth.test.jsx; e2e/account.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | New email |
| Input | Register with Shopper123 |
| Expected Output | Account created; signed in; returns to ?next |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-035-02: Lockout and duplicate email messages

| Field | Content |
|---|---|
| Test Case ID | TC-FR-035-02 |
| Test Case Name | Lockout and duplicate email messages |
| Related Requirement | FR035 |
| Related Test File | src/tests/auth/auth.test.jsx |
| Type | Integration (negative) |
| Priority | P2 |
| Preconditions | Locked account / taken email |
| Input | Sign in / register |
| Expected Output | Locked for 15 minutes message / email field error |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-035-03: Session survives reload; refresh reuse revokes sessions

| Field | Content |
|---|---|
| Test Case ID | TC-FR-035-03 |
| Test Case Name | Session survives reload; refresh reuse revokes sessions |
| Related Requirement | FR035, ADR-003 |
| Related Test File | backend tests/integration/auth/refresh.test.js; e2e/account.spec.js |
| Type | Integration + E2E |
| Priority | P1 |
| Preconditions | Signed-in customer |
| Input | Reload page; reuse an old refresh token |
| Expected Output | Still signed in; reuse gives 401 REFRESH_TOKEN_REUSE and revokes all sessions |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-OWASP-A01-01: Orders private to their owner

| Field | Content |
|---|---|
| Test Case ID | TC-OWASP-A01-01 |
| Test Case Name | Orders private to their owner |
| Related Requirement | OWASP A01 |
| Related Test File | backend tests/integration/orders/orderAccess.test.js |
| Type | Integration (negative) |
| Priority | P1 |
| Preconditions | Order of customer A |
| Input | GET as anonymous / customer B |
| Expected Output | 404 |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-NFR-008-01: No axe-core violations on main pages

| Field | Content |
|---|---|
| Test Case ID | TC-NFR-008-01 |
| Test Case Name | No axe-core violations on main pages |
| Related Requirement | NFR008 |
| Related Test File | src/tests/a11y/a11y.test.jsx |
| Type | Automated a11y |
| Priority | P2 |
| Preconditions | — |
| Input | Render home, catalog, product, auth, checkout, help, drawer |
| Expected Output | 0 violations (contrast checked manually) |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-NFR-009-01: No horizontal scroll on phones

| Field | Content |
|---|---|
| Test Case ID | TC-NFR-009-01 |
| Test Case Name | No horizontal scroll on phones |
| Related Requirement | NFR009 |
| Related Test File | e2e/catalog.spec.js |
| Type | E2E |
| Priority | P2 |
| Preconditions | Pixel 5 viewport |
| Input | Open /, /shop, /checkout, /sign-in |
| Expected Output | scrollWidth <= viewport |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |

## TC-FR-033-01: Offline chat never invents order data

| Field | Content |
|---|---|
| Test Case ID | TC-FR-033-01 |
| Test Case Name | Offline chat never invents order data |
| Related Requirement | FR033 |
| Related Test File | src/tests/chat/chat.test.jsx |
| Type | Integration |
| Priority | P2 |
| Preconditions | Chat server unreachable |
| Input | Ask "Where is my order 104?" |
| Expected Output | Points to Your orders; no status, carrier or tracking number |
| Actual Output | As expected |
| Status | Pass (2026-10-27) |
