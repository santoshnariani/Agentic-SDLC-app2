# [DATABASE] Admin Access

## User Story
As a system I need a RoleAssignment model so that Admin-only endpoints can distinguish an Admin from a Member without modifying the pre-built User model

## Assignment Order
Step 3 of 10 — assign after: [DATABASE] Perk Application is merged
Tier: DATABASE — extension slice

## Context
Pre-built models from copilot-instructions.md:
- User model in src/backend/prisma/schema.prisma — do not modify it
Previously added:
- Offer model and OfferStatus enum (database-offer-management)
- Redemption model (database-perk-application)
The User model carries no role and issued JWTs carry only `userId`, so FR-009 (Admin-level JWT) cannot be enforced today.
This issue adds only the role-assignment model required for this slice — it is additive and leaves User untouched.

## Models to Add
Enum: UserRole { MEMBER, ADMIN }

RoleAssignment
- id: Int @id @default(autoincrement())
- userId: Int @unique — id of the User this role belongs to
- role: UserRole @default(MEMBER) — Member or Admin

## Relationships
- RoleAssignment references one User by userId (one role per User)
- A User with no RoleAssignment row is treated as a MEMBER

## Seed Data
Add realistic sample data to src/backend/prisma/seed.ts so the app is
usable immediately after migration — never leave domain tables empty.

RoleAssignment — upsert by userId so re-seeding is idempotent:
- test@example.com → role ADMIN (look up the seeded test user's id; the existing Offer management and Redemption export UI and E2E journey run as this user and must keep working)

Seed data is required for:
- Frontend Offer and Redemption pages to keep loading for the test user once Admin enforcement lands
- Playwright tests to log in as an Admin and reach the Admin-only pages

## Acceptance Criteria
- [ ] Migration runs without errors
- [ ] RoleAssignment model created with userId (unique) and a UserRole enum (MEMBER/ADMIN) — not a String
- [ ] Seed grants test@example.com the ADMIN role, and re-running `npm run db:seed` does not duplicate it
- [ ] Pre-built User model and test user unchanged

## FR Coverage
- FR-009: RoleAssignment lets the backend tell an Admin JWT apart from a Member JWT
