# ADR-005: Customer Storefront Architecture

- **Status:** Proposed (for supervisor review within one sprint, SENG 34213 s1.3)
- **Date:** 2026-10-27
- **Tickets:** stackstorm-frontend-customer #109-#120; backend #100-#107
- **Related:** SRS FR001-FR016, FR023, FR027, FR035, NFR002, NFR008-NFR010; ADR-003 (authentication); ADR-004 (stock reservation)

## Context

An end-to-end audit on 2026-10-27, run against the real backend on PostgreSQL 16 and Redis, found that no order could be placed through the storefront:

- The storefront called routes that do not exist (`POST /cart`, `PUT /cart/:id`, `/auth/refresh-token`) and sent the guest cart id in a header the backend ignores. Every cart and wishlist call caught its own error and returned a fake success, so the cart only ever lived in `localStorage`.
- Catalog requests fell back to hard-coded `FALLBACK_PRODUCTS`, so shoppers could browse products that did not exist.
- The stock reservation used a hard-coded `variant_id: 1` and started a local 15-minute timer whenever the API failed.
- Views were switched by seven boolean flags in one 1,000-line component. Nothing had a URL, so refresh, the back button and shared links all returned to the home page.
- The access token was copied into three `localStorage` keys.
- `npm run lint`, `format:check` and `test:coverage` printed success messages without running any tool. A real lint run found 368 errors.
- In the backend, `GET /cart`, `POST /checkout/initiate`, `GET /orders/*` and `GET /returns/my-returns` all returned HTTP 500 because of column and association mismatches; payment could be "finalised" by any logged-in user; checkout priced every line at LKR 0.

## Decision

1. **The server is the source of truth.** Cart, wishlist, prices, stock, order totals and payment status are always read from the API after a change. The client never sends a price. Errors are shown to the shopper; nothing fails silently.
2. **Every page has a URL.** React Router 7 data router (`createBrowserRouter`). Catalog filters, search, sort and page live in the query string.
3. **One HTTP client and one error shape.** `src/lib/http.js` unwraps `{ success, data }` and converts the backend's three error formats into `ApiError { status, code, message, details }`. A single-flight `POST /auth/refresh` retries once on `TOKEN_EXPIRED`, because the backend rotates refresh tokens and treats reuse as theft (ADR-003).
4. **Token storage.** The access token is kept in memory only. The refresh token is the backend's httpOnly cookie, now scoped to `/api` so `/api/v1/auth/refresh` receives it. On load, the session is restored with one refresh call, made only if a non-secret `dfc.hasSession` hint exists.
5. **Guest identity.** Guest carts use a random `X-Session-Token` matching the backend rule. Guest orders are proven with the `guest_order_token` returned by checkout, sent as `X-Guest-Token`.
6. **Payment.** `POST /checkout/initiate` creates the order and the ADR-004 hold. The browser confirms the card with Stripe Elements; then `POST /payments/confirm` makes the server retrieve the PaymentIntent and check its status, amount and order id before it marks the order paid and confirms the reservation. Without a real Stripe key, the backend's mock provider is used, outside production only, and the UI labels it as test mode.
7. **State.** Four small React contexts (Auth, Cart, Wishlist, Toast) plus a `useResource` hook for page data. No global store library.
8. **Styling.** Plain CSS on the existing team tokens (`tokens.css`); one signature element (the handloom warp stripe).
9. **Quality gates.** ESLint (Airbnb, hooks, jsx-a11y) with zero warnings, Prettier, Vitest with an 80% coverage threshold, axe-core accessibility tests, Playwright E2E for the critical flows; `npm run verify` runs the CI set locally.

## Alternatives considered

| Option | Why not |
|---|---|
| Keep the existing single-component storefront and patch endpoints | No URLs, view flags and fake fallbacks are structural; patching would leave them |
| Redux Toolkit / React Query | Valid choices, but four contexts and one hook cover the needs with fewer dependencies for a 4-person team |
| Access token in `localStorage` | Readable by any injected script (OWASP A07); the refresh cookie already restores sessions |
| Client-side order totals | Prices and the free-delivery rule belong to the server (`checkout.service`); a client total can disagree with the charge |
| Mark orders paid from the browser after Stripe.js succeeds | The browser can be modified; only a server check of the PaymentIntent proves payment |

## Consequences

- The storefront works only with the backend fixes #100-#107 merged (merge order in the backlog).
- One extra request on load for returning visitors (session restore). First-time visitors make none.
- A page reload during the 15-minute hold loses the payment step; the hold then expires through the ADR-004 sweeper. The shopper can start checkout again.
- Product images in the seed data come from images.unsplash.com, so the CSP allows `img-src https:`. Tighten it once images are served from S3.
- The SDS must be updated to show the routes, the payment confirmation sequence and the token storage above.
