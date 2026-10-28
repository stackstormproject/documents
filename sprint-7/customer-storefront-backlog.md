# Sprint 7: Customer Storefront Backlog

Issue descriptions for the customer storefront work, in the SENG 34213 s4.3 template. Paste each section into a GitHub issue, set **Phase = Development**, the epic label, the priority and the **Sprint 7** milestone.

> Issue numbers continue from the last known issue (#99). If your board has moved on, renumber here and in the commit messages (`Closes #NN`) before opening the PRs.

## Summary and merge order

PRs are stacked: each branch starts from the one above it, so merge them into `develop` in this order (backend first).

| # | Repo | Branch | Title | Epic | Priority | Est. |
|---|---|---|---|---|---|---|
| 100 | backend | `fix/100-jwt-subject-identity` | Logged-in customers cannot use cart, merge or wishlist | epic:auth | P1 | 3 h |
| 101 | backend | `fix/101-cart-read-with-product` | GET /cart returns 500 and lacks product details | epic:api | P1 | 3 h |
| 102 | backend | `fix/102-checkout-pricing-and-order-lines` | Checkout fails and prices every line at LKR 0 | epic:core-features | P1 | 5 h |
| 103 | backend | `fix/103-order-reads-and-ownership` | Order reads return 500; orders readable by anyone | epic:security | P1 | 5 h |
| 104 | backend | `fix/104-payment-verification` | Verify payments server-side; remove unsafe finalize | epic:security | P1 | 8 h |
| 105 | backend | `feature/105-auth-refresh-logout` | Refresh-token rotation and logout endpoints | epic:auth | P1 | 6 h |
| 106 | backend | `fix/106-verified-purchaser-reviews` | Only verified purchasers may review | epic:core-features | P2 | 2 h |
| 107 | backend | `chore/107-seed-variants-for-all-products` | Seed purchasable variants and a category tree | epic:data-layer | P2 | 2 h |
| 109 | customer | `build/109-real-quality-gates` | Replace fake lint, format and coverage scripts | epic:ci-cd | P1 | 3 h |
| 110 | customer | `feature/110-api-client` | API client matching the backend contract | epic:integration | P1 | 6 h |
| 111 | customer | `feature/111-app-shell-and-routing` | URL routing, app shell and server-backed state | epic:ui-frontend | P1 | 8 h |
| 112 | customer | `feature/112-catalog-and-filters` | Shareable catalog with server-side filters | epic:ui-frontend | P1 | 6 h |
| 113 | customer | `feature/113-product-detail-and-reviews` | Product page with variant matrix and reviews | epic:ui-frontend | P1 | 6 h |
| 114 | customer | `feature/114-sign-in-and-register` | Sign in and registration pages | epic:auth | P1 | 4 h |
| 115 | customer | `feature/115-cart-drawer` | Server-backed bag drawer and cart page | epic:ui-frontend | P1 | 5 h |
| 116 | customer | `feature/116-checkout-and-payment` | Checkout with stock hold and verified payment | epic:core-features | P1 | 10 h |
| 117 | customer | `feature/117-account-orders-returns` | Account: orders, tracking, returns, addresses | epic:ui-frontend | P1 | 8 h |
| 118 | customer | `test/118-accessibility-and-responsive` | axe-core checks, responsive fixes | epic:testing | P2 | 3 h |
| 119 | customer | `test/119-e2e-critical-flows` | Playwright E2E for critical flows | epic:testing | P1 | 4 h |
| 120 | customer | `docs/120-readme-changelog-docker` | README, changelog, hardened Docker image, bundle split | epic:documentation | P2 | 3 h |
| 121 | documents | `docs/121-customer-storefront-pack` | ADR-005, backlog, test register, OWASP update | epic:documentation | P2 | 3 h |

Total: **99 h**. Sprint capacity is 160 h (documents/sprint-5/capacity.md); re-check against Sprint 7 planned hours before committing.

Raised but **not** done in this work: #122-#131 at the end of this file.

---

## Backend

### [DDP-#100] fix(auth): logged-in customers cannot use cart, merge or wishlist

**Labels:** epic:auth, P1-High, fix · **Estimate:** 3 h · **Dependencies:** none

#### User Story
As a signed-in customer, I want my bag and wishlist to belong to my account, so that they are there on any device.

#### Background / Context
- SRS Reference: FR005 (wishlist), FR006 (cart), NFR006 (cart persists for logged-in users)
- SDS Reference: Security design, auth middleware
- ADR Reference: ADR-003

Access tokens carry the subject as `userId`; `cartIdentity`, `cart.controller.mergeCart` and `wishlist.routes` read `req.user.id`. A logged-in customer gets `400 CART_IDENTITY_REQUIRED`, merge fails with `column "nan" does not exist`, and the wishlist is always empty. The existing merge test signed a hand-made `{ id }` token, which hid the bug.

#### Acceptance Criteria
**AC1: Customer cart**
Given a customer with a valid access token and no session token
When they call `POST /api/v1/cart/items`
Then the item is stored in a cart with their `customer_id` and the response is 201

**AC2: Guest cart merge**
Given a guest cart identified by `X-Session-Token`
When the customer calls `POST /api/v1/cart/merge` with that token
Then 200 is returned with `mergedCount` equal to the guest lines

**AC3: Admins have no cart**
Given a token with role `super_admin`
When it calls the merge endpoint
Then 401 is returned

**AC4: Wishlist**
Given a customer
When they toggle a product and then list the wishlist
Then the product is listed and `availableStock` is the real sum (not starting at 25)

**AC5: Guest wishlist**
Given no token
When `POST /wishlist/toggle` is called
Then 401 `AUTH_REQUIRED` is returned and nothing is stored

#### Technical Notes
- Normalise once in `middleware/auth.js` (`toRequestUser`: `id = userId ?? id ?? sub`).
- `cartIdentity` uses the shared `verifyAccessToken` (issuer check) and treats only role `customer` as a cart owner.

#### Test Requirements
- [x] Unit: `toRequestUser`, cart identity for customer / admin
- [x] Integration: AC1-AC5 (`tests/integration/customerIdentity.test.js`)
- [x] Fix `cartMerge.test.js` to sign a real access token

#### Definition of Done
- [x] All AC tests pass · [x] lint and format clean · [ ] PR to `develop` reviewed · [ ] CHANGELOG updated

---

### [DDP-#101] fix(cart): GET /cart returns 500 and lacks product details

**Labels:** epic:api, P1-High, fix · **Estimate:** 3 h · **Dependencies:** #100

#### User Story
As a shopper, I want my bag to show each item's name, picture, size and price, so that I know what I am buying.

#### Background / Context
- SRS Reference: FR006
- `cart.service.getCart` includes `ProductVariant` without `as: 'variant'`; Sequelize throws for every request.

#### Acceptance Criteria
**AC1: Cart lines**
Given a cart with one line
When `GET /api/v1/cart` is called
Then 200 is returned with `items[0]` containing `variant_id, quantity, unit_price, line_total, available_stock, size, color, sku` and `product { id, name, slug, primary_image, is_final_sale }`

**AC2: Price rule**
Given a product at LKR 4,500 and a variant surcharge of LKR 250
When the cart is read
Then `unit_price` is 4,750 (the rule checkout uses)

**AC3: Empty cart**
Given no cart
When the cart is read
Then `items` is an empty list

#### Test Requirements
- [x] Unit: `toCartLine` mapping and defaults
- [x] Integration: AC1 on PostgreSQL

#### Definition of Done
- [x] All AC tests pass · [x] lint clean · [ ] Postman collection updated with the response example

---

### [DDP-#102] fix(checkout): checkout fails and prices every line at LKR 0

**Labels:** epic:core-features, P1-High, fix · **Estimate:** 5 h · **Dependencies:** #101

#### User Story
As a shopper, I want checkout to create my order at the right price, so that I pay the advertised amount.

#### Background / Context
- SRS Reference: FR013, FR016, business rule: free delivery from LKR 10,000
- ADR Reference: ADR-004 (stock reservation)

`initiateCheckout` read `product.base_price`, `product.title` and `variant.price_override` (none exist) and inserted order items without `product_variant_id`, so every checkout returned 500. The reservation was never linked to its order, and a failed insert left stock held for 15 minutes.

#### Acceptance Criteria
**AC1: Priced order**
Given a variant of a LKR 4,200 product with 5 in stock
When a customer checks out 2 with standard delivery
Then 201 is returned with subtotal 8,400, shipping 450, total 8,850; the order and its lines are stored with `product_variant_id`; the reservation row has the order id and `reserved_quantity` is 2

**AC2: Free delivery**
Given a subtotal of LKR 10,000
When checking out with standard delivery
Then `shipping_fee` is 0

**AC3: Not enough stock**
Given 1 unit available
When 3 are requested
Then 409 is returned and nothing is reserved

**AC4: Failed insert releases the hold**
Given the order insert fails
When checkout runs
Then the reservation is released and the error is returned

**AC5: Validation**
Given a guest with no email, or `shipping_method: "teleport"`
When checkout is called
Then 400 is returned

#### Technical Notes
- Unit price = `products.price + product_variants.additional_price`; reject if not positive.
- Link the reservation inside the order transaction (`UPDATE stock_reservations SET order_id`).

#### Test Requirements
- [x] Unit: pricing with surcharge, missing price, unknown method, release on failure (mocks now use real column names)
- [x] Integration: AC1-AC3, AC5 (`tests/integration/checkout/initiate.test.js`)

#### Definition of Done
- [x] All AC tests pass · [x] coverage on `checkout.service` above 90% · [ ] PR reviewed

---

### [DDP-#103] fix(orders): order reads return 500 and orders are readable by anyone

**Labels:** epic:security, P1-High, fix · **Estimate:** 5 h · **Dependencies:** #102

#### User Story
As a customer, I want to see my orders, and only my orders, so that my purchases stay private.

#### Background / Context
- SRS Reference: FR009, FR010, FR023; business rules: 30-day returns, no final-sale returns
- OWASP A01 Broken Access Control

`getOrderById`, `getCustomerOrders` and `getCustomerReturns` selected `title`/`base_price` (they are `name`/`price`), so they returned 500. `getOrderById` also let an anonymous caller read any registered customer's order. `requestReturn` did not enforce the SRS return rules.

#### Acceptance Criteria
**AC1: History**
Given a customer with an order
When `GET /orders/my-orders` is called
Then 200 is returned with each line's `product_title`, `unit_price`, `product_slug`, `primary_image`, `is_final_sale`

**AC2: Ownership**
Given an order owned by customer A
When it is requested anonymously or by customer B
Then 404 is returned (not 403, so ids cannot be probed)

**AC3: Guest order**
Given a guest order
When it is requested with its guest token
Then 200 is returned; without the token, 404

**AC4: Return rules**
Given an order item
When a return is requested and the order is not delivered, or delivery was over 30 days ago, or the product is final sale
Then 400 is returned with the rule that failed

#### Test Requirements
- [x] Unit: `canReadOrder` matrix, `assertReturnEligible` boundaries (30 vs 31 days)
- [x] Integration: AC1-AC3 (`tests/integration/orders/orderAccess.test.js`)

#### Definition of Done
- [x] All AC tests pass · [ ] OWASP checklist A01 row updated (done in #121)

---

### [DDP-#104] fix(payment): verify PaymentIntent server-side and remove unsafe finalize

**Labels:** epic:security, P1-Critical, fix · **Estimate:** 8 h · **Dependencies:** #103

#### User Story
As the shop owner, I want an order to count as paid only when Stripe confirms the money, so that nobody gets goods without paying.

#### Background / Context
- SRS Reference: FR013, NFR011 (PCI DSS)
- ADR Reference: ADR-004, ADR-005
- OWASP A01, A04

`createPaymentIntent` used a `Payment` model that does not exist. `POST /orders/:id/finalize` let any logged-in user mark any order paid with an invented intent id; guests could not pay at all. The webhook could never verify signatures (the body was already JSON-parsed). Payment success confirmed the reservation by order id, so stock was never deducted.

#### Acceptance Criteria
**AC1: Intent for the owner only**
Given a `payment_pending` order
When its owner (customer token or guest `X-Guest-Token`) calls `POST /payments/create-intent`
Then 200 is returned with `provider`, `clientSecret`, `paymentIntentId` and the stored total; anyone else gets 404

**AC2: Server verification**
Given a Stripe PaymentIntent
When the owner calls `POST /payments/confirm`
Then the server retrieves the intent and marks the order `processing` only if status is `succeeded`, the amount equals the order total and `metadata.order_id` matches; otherwise 402

**AC3: Stock deducted**
Given a paid order with an active reservation
When payment is confirmed
Then the reservation becomes CONFIRMED and `stock_quantity` drops by the ordered quantity

**AC4: Idempotent**
Given an order already `processing`
When confirm or the webhook runs again
Then nothing changes and 200 is returned

**AC5: Forged intent**
Given an intent id that is not the order's
When confirm is called
Then 400 is returned

**AC6: Finalize removed**
When `POST /orders/:id/finalize` is called
Then 404 is returned

**AC7: Webhook**
Given a signed `payment_intent.succeeded` event with a matching amount
When Stripe calls the webhook
Then the order is marked paid; a mismatched amount is ignored

**AC8: Mock provider**
Given no real `STRIPE_SECRET_KEY` and `NODE_ENV` not production
When an intent is created
Then provider `mock` is used; in production without a key, 503

#### Technical Notes
- `express.raw` on the webhook paths before `express.json`.
- Idempotency key `order-<id>-<amount>` on intent creation; reuse an open intent.

#### Test Requirements
- [x] Unit: provider selection, minor units, ownership, 402/400/409 paths, webhook cases
- [x] Integration: pay and deduct stock, guest pay, foreign caller 404, forged intent 400, finalize 404 (`tests/integration/checkout/payment.test.js`)
- [ ] Manual: Stripe test mode with `stripe listen --forward-to` on staging

#### Definition of Done
- [x] All AC tests pass · [ ] security review by a second team member (s4.3.1) · [ ] Postman updated

---

### [DDP-#105] feat(auth): refresh-token rotation and logout

**Labels:** epic:auth, P1-High, feat · **Estimate:** 6 h · **Dependencies:** #104

#### User Story
As a logged-in customer, I want my session to stay active without signing in again, so that checkout is not interrupted (guideline s4.3.1 example).

#### Background / Context
- SRS Reference: FR035, NFR010
- ADR Reference: ADR-003 (JWT with rotating refresh tokens)

Access tokens expire after 1 h; there was no refresh or logout endpoint, and the refresh cookie path `/api/auth` was never sent to `/api/v1/auth`.

#### Acceptance Criteria
**AC1: Rotation**
Given a valid refresh cookie
When `POST /api/v1/auth/refresh` is called
Then 200 is returned with a new access token and the customer, and the cookie is replaced

**AC2: Reuse = theft**
Given a refresh token that was already rotated
When it is presented again
Then 401 `REFRESH_TOKEN_REUSE` is returned and every live session of that user is revoked

**AC3: Expired / missing**
Then 401 `REFRESH_TOKEN_EXPIRED` or `REFRESH_TOKEN_MISSING`

**AC4: Logout**
When `POST /auth/logout` is called
Then 204 is returned, the token is revoked and the cookie cleared

**AC5: Cookie scope**
Then the cookie is `HttpOnly`, `SameSite=Strict`, `Path=/api`

#### Test Requirements
- [x] Integration: AC1-AC5 (`tests/integration/auth/refresh.test.js`)

---

### [DDP-#106] fix(review): only verified purchasers may review

**Labels:** epic:core-features, P2-Medium, fix · **Estimate:** 2 h · **Dependencies:** #105

**User Story:** As a shopper, I want reviews to come from real buyers, so that I can trust them.
**Background:** SRS business rule "Only customers who have purchased a product may submit a review"; FR008.

**AC1** Given a customer with no paid order containing the product, When they post a review, Then 403 is returned and nothing is stored.
**AC2** Given only `confirmed/processing/shipped/delivered` orders, When checking purchase, Then unpaid or cancelled orders do not count.

**Tests:** [x] unit 403 and status filter.

---

### [DDP-#107] chore(seed): seed purchasable variants and a category tree

**Labels:** epic:data-layer, P2-Medium, chore · **Estimate:** 2 h · **Dependencies:** #106

**User Story:** As a developer or panel member, I want every seeded product to be buyable, so that the demo covers the whole catalogue.
**Background:** 6 of 13 seeded products had no variant (SKU collisions swallowed by `.catch(() => {})`); categories were created with snake_case keys, so the tree was flat.

**AC1** Given a fresh database, When `npm run db:seed` runs, Then every product has at least one variant with SKU `DFC-<productId>-<size>`.
**AC2** Then apparel has S, M, L, XL (XL +LKR 250, low stock), sarees Free Size, accessories One Size.
**AC3** Then `GET /categories` returns Men > Batik Shirts and the other sub-categories.
**AC4** Running the seed twice creates no duplicates.

**Tests:** [x] unit size matrix and key mapping.

---

## Customer storefront

### [DDP-#109] build(customer): replace fake lint, format and coverage scripts

**Labels:** epic:ci-cd, P1-High, build · **Estimate:** 3 h · **Dependencies:** none

#### User Story
As the team, we want CI to fail when the code breaks our standards, so that the green badge means something.

#### Background / Context
- Guideline s5.2, s7.2, s8.2; documents/standards/coding-standards.md
- `lint` and `format:check` printed "0 errors"; `test:coverage` wrote a hand-made `coverage/index.html`. A real lint run found 368 errors. `.env` was committed.

#### Acceptance Criteria
**AC1** Given a lint error, When `npm run lint` runs, Then it exits non-zero (ESLint Airbnb + hooks + jsx-a11y + Prettier, `--max-warnings 0`).
**AC2** Given unformatted code, When `npm run format:check` runs, Then it exits non-zero.
**AC3** Given coverage below 80%, When `npm run test:coverage` runs, Then it fails and writes html, lcov and json-summary reports.
**AC4** `.env` is untracked and ignored; `.env.example` documents every variable with placeholders.

#### Test Requirements
- [x] Run each script on a deliberately broken file and confirm it fails

---

### [DDP-#110] feat(customer): API client matching the backend contract

**Labels:** epic:integration, P1-High, feat · **Estimate:** 6 h · **Dependencies:** #109, backend #100-#105

#### User Story
As a shopper, I want what I see to be what the shop actually has, so that I never order something that does not exist.

#### Background / Context
- SRS: FR001-FR016; ADR-005 decisions 1, 3, 4, 5
- Legacy calls hit non-existent routes and caught their own errors with fake successes.

#### Acceptance Criteria
**AC1: Contract** Given each API module, When it is called, Then it uses the verified route, verb and body (one contract test per endpoint).
**AC2: Errors** Given any backend error shape, When a call fails, Then an `ApiError { status, code, message, details }` is thrown; a network failure gives `NETWORK_ERROR` and a readable message.
**AC3: Refresh** Given an expired access token, When two requests fail with `TOKEN_EXPIRED` at once, Then exactly one `POST /auth/refresh` is made and both requests are retried once.
**AC4: Tokens** Then the access token is never written to `localStorage`.
**AC5: Guests** Then guest cart calls send `X-Session-Token`; guest order calls send `X-Guest-Token`.
**AC6: Prices** Then checkout sends only variant ids and quantities.

#### Test Requirements
- [x] 63 tests: `src/tests/lib/*`, `src/tests/api/endpoints.test.js`

---

### [DDP-#111] feat(customer): URL routing, app shell and server-backed state

**Labels:** epic:ui-frontend, P1-High, feat · **Estimate:** 8 h · **Dependencies:** #110

#### User Story
As a shopper, I want every page to have its own address, so that refresh, the back button and shared links work.

#### Background / Context
- SRS 12.1 (persistent header with logo, search, cart, account), NFR007
- ADR-005 decisions 2 and 7

#### Acceptance Criteria
**AC1** Given any page, When it is refreshed or opened from a link, Then the same page loads.
**AC2** Given a returning customer, When the app loads, Then the session is restored from the refresh cookie; first-time visitors make no refresh request.
**AC3** Given a guest with items who signs in, Then the guest cart is merged once into the customer cart.
**AC4** Given a guest opening `/account/...`, Then they are redirected to `/sign-in?next=...`.
**AC5** The header shows search, top-level categories from `/categories`, bag and wishlist counts; a skip link reaches the main content.
**AC6** While the chat server is offline, the chat never invents order statuses or agents.

#### Test Requirements
- [x] Provider tests (auth, cart, wishlist, toasts), shell tests, chat and socket tests

---

### [DDP-#112] feat(catalog): shareable catalog with server-side filters

**Labels:** epic:ui-frontend, P1-High, feat · **Estimate:** 6 h · **Dependencies:** #111

#### User Story
As a shopper, I want to filter by category, size and price and share the result, so that I can find and send the right clothes.

#### Background / Context
- SRS: FR001, FR002, FR003; UC-002

#### Acceptance Criteria
**AC1** Given `/shop/men-batik-shirts?size=M&price=3000-6000`, Then the API is called with those filters and the controls show them as selected.
**AC2** Given any filter change, Then the URL updates and the page returns to 1.
**AC3** Each card shows price and the FR003 label: In stock / Only X left (X ≤ 5) / Out of stock.
**AC4** No results: a clear-filters action; API failure: a retry; never fallback products.
**AC5** On phones, filters sit behind a "Filters (n)" button.

#### Test Requirements
- [x] 20 tests in `src/tests/catalog`; E2E `catalog.spec.js`

---

### [DDP-#113] feat(pdp): product page with variant matrix and verified reviews

**Labels:** epic:ui-frontend, P1-High, feat · **Estimate:** 6 h · **Dependencies:** #112

#### User Story
As a shopper, I want to see each size's price and stock before I add it, so that I do not pick something unavailable.

#### Background / Context
- SRS: FR003, FR004, FR008; UC-003 step 1

#### Acceptance Criteria
**AC1** Sold-out sizes are visible but disabled; the price follows the selected variant (XL surcharge).
**AC2** Quantity cannot exceed available stock or 10.
**AC3** Add to bag without a size asks for one; a 409 says how many are left.
**AC4** No reviews: "No reviews yet" (not the backend's default 5.0). Signed-in customers can publish a review; 403 and 409 are explained.
**AC5** Unknown slug shows the not-found page.

#### Test Requirements
- [x] 19 tests in `src/tests/product`

---

### [DDP-#114] feat(auth): sign-in and registration pages

**Labels:** epic:auth, P1-High, feat · **Estimate:** 4 h · **Dependencies:** #113

#### User Story
As a customer, I want to register and sign in, so that I can track orders and save addresses (FR035, UC-001).

#### Acceptance Criteria
**AC1** Client checks mirror the server rules; field errors are announced to screen readers.
**AC2** 401 wrong credentials, 423 locked (15 min), 429 rate limited and duplicate email each get a plain message on the right field.
**AC3** `?next=` returns the user after sign-in; only same-site paths are accepted.
**AC4** No "Forgot password" link until the backend supports reset (#123).

#### Test Requirements
- [x] 25 tests in `src/tests/auth`

---

### [DDP-#115] feat(cart): server-backed bag drawer and cart page

**Labels:** epic:ui-frontend, P1-High, feat · **Estimate:** 5 h · **Dependencies:** #114

#### User Story
As a shopper, I want to review and change my bag quickly, so that I check out with the right items (FR006).

#### Acceptance Criteria
**AC1** The drawer is a modal dialog: focus moves in, Tab stays inside, Escape closes, focus returns to the bag button.
**AC2** Quantity changes and removals go to the server and the bag is re-read.
**AC3** A line asking for more than is available is flagged and checkout is disabled.
**AC4** Shows progress to free delivery at LKR 10,000.

#### Test Requirements
- [x] 11 tests in `src/tests/cart`

---

### [DDP-#116] feat(checkout): checkout with 15-minute hold and verified payment

**Labels:** epic:core-features, P1-Critical, feat · **Estimate:** 10 h · **Dependencies:** #115, backend #102, #104

#### User Story
As a shopper, I want to pay safely and know my items are held while I pay, so that nobody else buys them first (UC-003, FR013, FR014, FR016).

#### Background / Context
- ADR-004 (15-minute hold), ADR-005 decision 6

#### Acceptance Criteria
**AC1: Details** Guests give an email; customers can use a saved address (default preselected) or a new one and save it; 5-digit postcode required; delivery or collection.
**AC2: Hold** Given valid details, When "Continue to payment" is pressed, Then the order and hold are created and a countdown from 15:00 is shown; totals come from the server.
**AC3: Expiry** Given the countdown reaches 0, Then the hold is released and the shopper is told; "Change details" also releases it.
**AC4: Payment** With a Stripe key, the card is confirmed by Stripe, then `POST /payments/confirm`; without one, a labelled test-mode button. Declines and verification failures are shown and can be retried.
**AC5: Stock conflict** A 409 at initiate refreshes the bag and explains it.
**AC6: Confirmation** Success opens `/orders/:id/confirmation`, readable by guests with their token; paid lines leave the bag.

#### Test Requirements
- [x] 22 tests in `src/tests/checkout`; E2E guest checkout
- [ ] Manual on staging with Stripe test card 4242 4242 4242 4242 and a decline card 4000 0000 0000 0002

---

### [DDP-#117] feat(account): account area with orders, tracking and returns

**Labels:** epic:ui-frontend, P1-High, feat · **Estimate:** 8 h · **Dependencies:** #116, backend #103

#### User Story
As a customer, I want to see my orders, follow delivery and return items, so that I can manage purchases myself (FR009, FR010, FR011, FR023, FR027, UC-004, UC-005).

#### Acceptance Criteria
**AC1** Orders list with pages; detail with tracking timeline, current step marked.
**AC2** "Return or exchange" appears only when allowed (delivered, ≤ 30 days, not final sale, not already requested); exchanges list other in-stock sizes.
**AC3** Address book: add, edit, make default, delete with confirmation.
**AC4** Profile edit; loyalty tier and points.
**AC5** Sign-out returns to the home page.

#### Test Requirements
- [x] 34 tests in `src/tests/account`; E2E customer flow

---

### [DDP-#118] test(a11y): axe-core checks and responsive fixes

**Labels:** epic:testing, P2-Medium, test · **Estimate:** 3 h · **Dependencies:** #117

**User Story:** As a shopper using a screen reader or a phone, I want the shop to work for me (NFR008, NFR009).
**AC1** axe-core reports no violations on the main pages, the open drawer and a form with errors.
**AC2** No horizontal scrolling at 360 px on catalog, product and checkout.
**Tests:** [x] `src/tests/a11y`; [ ] manual colour-contrast audit (jsdom cannot compute it).

---

### [DDP-#119] test(e2e): Playwright suite for the critical flows

**Labels:** epic:testing, P1-High, test · **Estimate:** 4 h · **Dependencies:** #118

**User Story:** As the team, we want the top three shopping flows tested end to end, so that a deploy cannot silently break buying (guideline s6.2).
**AC1** Guest checkout, customer purchase and tracking, and guest-bag merge pass against a running backend.
**AC2** `E2E_BASE_URL` selects the environment; a nightly workflow runs against staging and uploads the report.

---

### [DDP-#120] docs(customer): README, changelog, hardened Docker image

**Labels:** epic:documentation, P2-Medium, docs · **Estimate:** 3 h · **Dependencies:** #119

**AC1** README meets s9.3: description, prerequisites, env vars, fresh-machine steps, deployed link, CI and coverage badges.
**AC2** Docker uses `npm ci`, VITE_* build args, and nginx keeps security headers on every location.
**AC3** Checkout, account and chat load as separate chunks.
**Open:** run `docker run --rm <image> nginx -t` once (not possible in the build environment).

---

### [DDP-#121] docs: storefront ADR, backlog, test register, OWASP update

**Labels:** epic:documentation, P2-Medium, docs · **Estimate:** 3 h

**AC1** ADR-005 records the architecture and is reviewed with the supervisor within one sprint.
**AC2** The test case register lists the storefront functional test cases (s6.3.3).
**AC3** The OWASP checklist points to real files and tests.

---

## Raised, not done (create these issues)

| # | Repo | Title | Why it matters | Priority |
|---|---|---|---|---|
| 122 | admin + backend | Remove the `mock-admin-token` / `jwt-admin-token-xyz` backdoor | Any request with that literal token is `super_admin` whenever `NODE_ENV` is not production, which includes staging (OWASP A01/A07). The admin app's login depends on it, so fix both together | P1-Critical |
| 123 | backend | Password reset by email | FR035 requires recovery; no route exists, so the storefront shows no "Forgot password" link | P1 |
| 124 | backend | Transactional emails (order confirmation, shipping, return updates) | FR012; nothing sends email, so the UI promises none | P2 |
| 125 | backend | Clear the cart server-side when an order is paid | The storefront removes paid lines itself; the server should own this | P2 |
| 126 | backend | Product detail cache shows stale stock for up to 1 h | `getProductBySlug` caches `available_stock` in Redis for 3,600 s; invalidate on reserve/confirm/release (FR003 "real-time") | P2 |
| 127 | backend | Unreviewed products report a 5.0 average | `getProductRatingStats` defaults to 5.0; return null | P3 |
| 128 | backend | Cancel orders pending payment for more than 48 h | SRS business rule; no job exists | P2 |
| 129 | backend | Award loyalty points on paid orders | FR027 accrual; there is no earning function | P2 |
| 130 | backend | Order tracking invents a carrier and tracking number | `getOrderTracking` returns "PromptX Express / Koombiyo Logistics" and `TRK-DFC-<id>` when none is set; return null instead | P3 |
| 131 | infra | Coverage upload and E2E stage in the shared workflow | Codecov upload for the README badge; a docker-compose E2E job | P3 |
