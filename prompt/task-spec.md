 # Product Specification: Pocketful (Self-Contained Ledger & Wallet System)

## Project Overview
Build a high-reliability, clean-room clone of a digital wallet and ledger service. The system must process concurrent financial transactions, maintain strict double-entry accounting integrity, enforce zero-float precision, and run completely offline in an isolated container without outbound network access.

---

## Core System Architecture Constraints
1. **Zero External Dependencies at Runtime:** SQLite (`better-sqlite3` in WAL mode) as local database. No cloud DBs (Supabase), no external payment APIs, no network access.
2. **Integer Math (Zero-Float Guarantee):** All monetary values must be stored and calculated strictly in **cents/integer units**. Float and double types are forbidden for balances or amounts.
3. **Double-Entry Ledger:** Balance is a derived state from immutable debit/credit ledger entries. No mutable `balance` column without backing transaction history.
4. **Isolated Container Execution:** Must compile and run via Docker with `--network=none` flag, serving both API and static UI dashboard on port 3000.

---

## Stage-by-Stage Feature Deliverables

### Stage 1: Base Wallet Engine & CRUD MVP
- SQLite database schema: `users`, `accounts`, `ledger_entries`, `idempotency_records`.
- REST endpoints:
  - `POST /api/accounts`: Create user wallet with initial deposit.
  - `GET /api/accounts/:id/balance`: Fetch current derived balance and account status.
  - `POST /api/transfers`: Execute funds transfer between two accounts.
- Web Dashboard (Vite + React) showing active accounts, current balances, and transaction history.
- Verification: Docker multi-stage build that serves the app offline.

### Stage 2: Concurrency Guards & Invariant Verifications
- Transaction isolation (`BEGIN IMMEDIATE` in SQLite transactions).
- Strict validation rules:
  - **Invariant 1 (Conservation of Money):** Sum of all balances across system remains constant during transfers. Money can never be created or destroyed.
  - **Invariant 2 (Non-Negative Balance):** Standard accounts cannot drop below 0 cents unless explicit credit limit is flagged.
- Property-based testing with `fast-check` simulating parallel transfers.
- **Abuse Mode UI:** Interactive button on dashboard firing 100+ concurrent transfer requests to validate zero double-spending or state divergence.

### Stage 3: Idempotency, Retries & Fair Splitting
- Header-based Idempotency (`X-Idempotency-Key`): Retrying the exact same request $N$ times executes the transaction exactly once and returns cached response.
- **Fair Splitting Algorithm:** Split an bill/amount among $K$ users with deterministic integer rounding (remainder cents allocated fairly and reproducibly).
- Ledger Inspector UI tab: Complete step-by-step trace breakdown for any account balance.

### Stage 4: Extension & Self-Healing Regression Gate
- Extend system with scheduled/recurring transfers capability.
- Maintain regression suite: extending features must not break existing invariant certificates or double-entry audit checks.

---

## Acceptance & Quality Checklist for Reviewer
- [ ] Automated property tests (`fast-check`) pass green for 10,000 runs.
- [ ] Docker container boots and responds on HTTP in zero-network environment.
- [ ] `EVIDENCE.md` updated with Invariant Certificates and token/resource usage logs.
