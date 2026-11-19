# Draft API contract
All endpoints accept a seeded demo identity in trusted environment; backend checks permissions and scope. DTOs and error codes must be finalized before implementation.
- GET /demo/users — list seeded identities.
- GET /work-orders — list filtered by actor role and scope.
- POST /work-orders — ST-only create, type INS/REP/REM, customer_id, equipment_id; auto-assign technician; enforce active uniqueness.
- GET /work-orders/{id} — details, photos, approvals, chronological events.
- POST /work-orders/{id}/cancel — creator ST before initial approval.
- POST /work-orders/{id}/initial-review — team SSM approve/reject (reject requires reason).
- POST /work-orders/{id}/complete — assigned technician, at least one JPEG photo and explanation, after initial approval.
- POST /work-orders/{id}/st-review — scoped ST approve/reject (reject requires reason).
- POST /work-orders/{id}/final-review — scoped SSM approve/reject (reject requires reason).
All mutations are atomic and idempotent; unauthorized access and invalid transitions return explicit errors.