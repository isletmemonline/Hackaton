# Draft API contract
All endpoints accept a seeded demo identity in a trusted local environment; backend resolves identity and checks role and scope. This is not production authentication. DTOs and error codes remain to be finalized before implementation.

## Authorization invariants
- ST creation: selected customer must be assigned to the acting ST; selected equipment must belong to that customer.
- ST cancellation and completion review: acting ST must equal work_orders.creator_st_id. Cancellation only before initial SSM approval; completion review only after assigned technician completion.
- Initial and final SSM review: acting SSM must be authorized for the creator ST's team. Initial review only while OPEN; final review only after creator ST approved completion.
- Technician completion: acting technician must equal work_orders.assigned_technician_id, initial SSM approval must already exist, and current status must be PENDING.
- The API enforces these rules on every action; client-side filtering is not sufficient.

## Endpoints
- GET /demo/users — list seeded identities.
- GET /work-orders — list filtered by actor role and scope.
- POST /work-orders — ST-only create, type INS/REP/REM, customer_id, equipment_id; validate customer/equipment/ST relationships; auto-assign technician; enforce active uniqueness.
- GET /work-orders/{id} — details, photos, approvals, chronological events, scoped to actor.
- POST /work-orders/{id}/cancel — creator ST before initial approval; sets CANCELLED.
- POST /work-orders/{id}/initial-review — authorized team SSM approve (OPEN → PENDING) / reject (OPEN → CANCELLED, reason required).
- POST /work-orders/{id}/complete — assigned technician, at least one JPEG photo and explanation, after initial approval (PENDING → COMPLETED).
- POST /work-orders/{id}/st-review — creator ST approve (remain COMPLETED, advance approval stage) / reject (CANCELLED, reason required).
- POST /work-orders/{id}/final-review — authorized team SSM approve (COMPLETED → CLOSED) / reject (COMPLETED → CANCELLED, reason required), only after ST approval.

## State and consistency
Operational statuses: OPEN, PENDING, COMPLETED, CLOSED, CANCELLED. CLOSED and CANCELLED terminal; other statuses active. Approval stages are separate and ordered INITIAL_SSM → COMPLETION_ST → FINAL_SSM. All mutations are atomic and idempotent; unauthorized access and invalid transitions return explicit errors. Reject duplicate conflicting decisions. Store reviewer identity and timestamp in audit history.