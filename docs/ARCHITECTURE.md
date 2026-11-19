# Architecture
Expo React Native TypeScript mobile and Next.js TypeScript Backoffice both consume a Python HTTP API. API implements all business rules, identity/role checks, work-order state machine, technician assignment, photo validation and audit. SQLite data/asset.sqlite is private to backend; data/photos contains uploaded JPEGs. Demo identity selection is not production authentication. No Supabase or direct database access from clients. maya-scss is authoritative for design tokens when supplied. Use API DTOs aligned with API-CONTRACT.md.

## Authorization and consistency
- Acting demo identity is resolved by Python API; never trust a role supplied by a client.
- Only ST creates orders for assigned customers; equipment must belong to that customer.
- Creator ST alone may cancel before initial approval or review technician completion.
- Initial and final SSM decisions require authorization for creator ST's team.
- Assigned technician can complete only after initial SSM approval.
- All status transitions, approval records, event records, and one-active-order-per-equipment checks are enforced transactionally in backend/SQLite.
- Operational statuses: OPEN, PENDING, COMPLETED, CLOSED, CANCELLED; CLOSED and CANCELLED terminal. Approval stages remain separate.