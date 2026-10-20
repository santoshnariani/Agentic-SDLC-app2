# [BACKEND] Admin Access

## User Story
As an Admin I want Offer management and Redemption export to be restricted to Admins so that Members cannot configure discounts or download the Redemption ledger

## Assignment Order
Step 7 of 10 — assign after: [BACKEND] Redemption Export is merged
Tier: BACKEND — extension slice

## Context
Pre-built from copilot-instructions.md:
- JWT auth middleware → src/backend/middleware/auth.ts (`authenticate` sets `req.userId`) — do not modify it
- Auth routes → src/backend/routes/auth.ts — do not modify; tokens stay `{ userId }`
Previously added:
- RoleAssignment model and UserRole enum (database-admin-access)
- Offer endpoints (backend-offer-management)
- Transaction endpoint (backend-perk-application)
- Redemption endpoints (backend-redemption-export)
This issue adds a new `requireAdmin` middleware file next to the pre-built auth middleware, and explicitly requires
adding it to the existing Offer and Redemption route files. No new endpoints are added.

requireAdmin runs after `authenticate`:
1. Look up the RoleAssignment for `req.userId`
2. role ADMIN → call next()
3. Missing row or role MEMBER → 403 `{ error: "Forbidden — admin role required" }`

## API Endpoints
Apply `authenticate` then `requireAdmin` to:
- POST /api/offers, GET /api/offers, PATCH /api/offers/:id
- GET /api/redemptions, GET /api/redemptions/export

Leave unchanged (any authenticated Member may transact):
- POST /api/transactions — `authenticate` only

## Acceptance Criteria
- [ ] Every Admin endpoint listed above returns 401 `{ error: string }` without a valid JWT
- [ ] Every Admin endpoint returns 403 `{ error: string }` for a valid JWT whose user has no ADMIN role (e.g. a newly registered user)
- [ ] Every Admin endpoint returns its normal success response for the seeded Admin (test@example.com)
- [ ] POST /api/transactions still accepts a valid non-Admin JWT

## FR Coverage
- FR-009: Admin endpoints require an Admin-level JWT; non-Admins get 403 and unauthenticated requests get 401
- FR-008: Member transaction submission stays open to any authenticated user
