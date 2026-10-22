# ADR-004: Stock Reservation with Row Locks and a Database Ledger

- **Status:** Accepted
- **Date:** 2026-10-22
- **Ticket:** backend#46
- **Related:** SRS FR-<inventory>, NFR-<concurrency>; SDS Section <X.Y>

## Context
Checkout must hold stock for 15 minutes without overselling. Three naive approaches fail:

- An unprotected read-then-write lets two shoppers take the last item.
- Holding a database transaction open for 15 minutes exhausts the connection pool.
- In-memory timers or Redis-only holds lose state on restart, and Redis key expiry never decrements `reserved_quantity`, so stock stays stuck as "reserved".

## Decision
- Reserve inside a short transaction (milliseconds) using `SELECT ... FOR UPDATE` on each variant row. Increment `reserved_quantity` only if `stock_quantity - reserved_quantity >= requested`.
- Lock variants in ascending id order so carts like [A,B] and [B,A] cannot deadlock.
- Record every reservation in `stock_reservations` and `reservation_items` in the same transaction. The database ledger is the source of truth.
- Store a copy in Redis with a 900 second TTL as a fast-path cache only. A Redis failure after commit is logged and ignored.
- Add a CHECK constraint (`0 <= reserved_quantity <= stock_quantity`) as the last line of defence.
- A database-backed sweeper job (backend#48) releases expired reservations.

## Alternatives Considered
- **Optimistic locking (version column):** high retry rate during flash sales.
- **Long-lived database transactions:** exhaust the connection pool.
- **Redis-only reservations or `setTimeout`:** lose state on restart and leave stock stuck.

## Consequences
- Overselling is prevented by the database itself.
- Two extra tables and a sweeper job must be maintained.
- Admin stock edits cannot lower `stock_quantity` below the reserved amount.
- Redis outages degrade only the cache, not correctness.