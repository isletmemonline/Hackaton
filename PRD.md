# Asset MVP — Product Requirements Document
Version: 1.1 | 19 November 2026 | Two-day hackathon

## Goal
Manage one-equipment installation (INS), repair (REP), and removal (REM) work orders through ST, SSM, and technician roles, using mobile and backoffice clients.

## Stack
- Expo / React Native / TypeScript mobile
- Next.js / TypeScript backoffice
- Python HTTP API; business rules and authorization centralized here
- SQLite database in `data/`
- `maya-scss/` is the authoritative styling source. Its contents will be supplied later. Do not invent brand colors, typography, or component standards; adapt source tokens to React Native.

## Roles and access
- Only ST creates work orders for customers assigned to that ST. The selected equipment must belong to the selected customer.
- The work-order creator ST is the responsible ST and is the only ST permitted to approve/reject technician completion or cancel before initial approval.
- Initial and final SSM approval/rejection belongs to an SSM authorized for the creator ST's team. Team membership must be verified server-side at decision time.
- Technician works only on assigned, initially SSM-approved orders.
- The Python API verifies the selected seeded demo identity, its role, team/customer relationship, and the current approval step on every read/write. Client UI filtering is not authorization.
- Demo-only identity selection from backend-defined ST, SSM, and technician accounts; no production login. Demo identity mode must not be deployed publicly.

## Work orders
- One `work_orders` table with type `INS|REP|REM`; optional type-specific detail tables.
- Exactly one equipment per order.
- Operational statuses: `OPEN`, `PENDING`, `COMPLETED`, `CLOSED`, `CANCELLED`. `CLOSED` and `CANCELLED` are terminal. Every other status is active.
- Only one active order per equipment, enforced in API and SQLite with a conditional unique index. Technician completion remains active until final closure or cancellation.
- Orders cannot be edited after creation.
- Before initial SSM approval, creator ST may cancel their order and open a new one.
- All rejections, including initial SSM, ST completion, and final SSM, result in `CANCELLED`; reason required. Preserve immutable event history. Rejected orders are not reopened.
- Approval stages are separate from operational status. `OPEN` is created awaiting initial SSM review; `PENDING` follows initial SSM approval and permits technician work; `COMPLETED` persists during both completion approvals; final approval sets `CLOSED`.

## Technician assignment and execution
- Assign automatically at creation. MVP routes all orders to one seeded technician.
- Future design resolves technician from customer region; ST and SSM never select technicians manually.
- Assigned order becomes actionable only after initial SSM approval.
- No technician acceptance stage.
- Completion requires at least one photo and nonblank explanation.
- Technician evidence is read-only for ST and SSM.
- Completion sets operational status `COMPLETED`; approval state tracked separately, without `PENDING_ST_APPROVAL` operational status.

## Approval workflow
1. Creator ST creates; backend assigns technician; status `OPEN`.
2. SSM authorized for the creator ST's team initially approves (`PENDING`) or rejects (`CANCELLED` with mandatory reason).
3. On approval, assigned technician can work and submit mandatory evidence; status `COMPLETED`.
4. The creator ST reviews completion: approve → final SSM review, status remains `COMPLETED`; reject → `CANCELLED`.
5. SSM authorized for the creator ST's team performs final review: approve → `CLOSED`; reject → `CANCELLED`.
- Approval notes optional; rejection reasons mandatory.
- All decisions record actor, timestamp, step, reason, and outcome.
- No step may be skipped or performed twice with conflicting outcomes.

## Photo storage
- SQLite at `data/asset.sqlite`; files in sibling directory `data/photos/`.
- Filenames `<work_order_code>_1.jpg`, `<work_order_code>_2.jpg`, etc. JPEG default; server normalizes uploads and prevents collisions/path traversal.
- Database stores relative file paths and metadata, not photo blobs.
- Photo paths served via backend endpoints with role checks.

## Screens
- Demo role/user selection.
- ST: customer/equipment selection, create, cancel preapproval, own orders, completion review.
- SSM: initial and final approval queues scoped to team, order detail.
- Technician: actionable assignments, completion with photos and explanation.
- Shared order detail: single chronological timeline of events, decisions, and evidence.

## Out of scope
- Inventory transfers and equipment movement accounting
- Production-grade authentication
- Multiple-technician scheduling and regional assignment UI
- Payment and unrelated business processes

## Acceptance checks
- Only creator ST can review completion or cancel before initial approval; unrelated STs cannot act.
- SSM outside creator ST's team cannot perform initial/final reviews.
- ST creation validates assigned customer and equipment/customer relationship.
- Unauthorized role cannot create, approve, or view out-of-scope orders.
- Concurrent creation cannot create two active orders for same equipment.
- Technician cannot complete before initial SSM approval.
- Completion blocked without photo or explanation.
- Rejection requires reason and cancels order.
- Completion evidence cannot be modified by ST/SSM.
- Timeline accurately reflects each step; repeated approval request is idempotent.
- No styling assumptions if Maya SCSS contents are absent.

## Explicit implementation assumptions to validate
- Order code format `WO-000123` is illustrative; stable unique code generation required.
- Demo role selection is a convenience for a local trusted environment, not security authentication.
- Detailed DTO fields, upload size limits, and error codes remain draft pending API contract review.