# Data model
- users: id, name, role (ST/SSM/TECHNICIAN), team_id, region_id. ST and SSM team association supports server-side authorization.
- customers: id, name, assigned_st_id, region_id. assigned_st_id references responsible ST.
- equipment: id, customer_id, external_ref. equipment.customer_id must equal selected work-order customer_id.
- work_orders: id, order_no, type (INS/REP/REM), equipment_id, customer_id, creator_st_id, assigned_technician_id, operational_status, explanation, created_at, completed_at, closed_at. creator_st_id is the responsible ST for completion review and preapproval cancellation.
- approvals: id, work_order_id, stage (INITIAL_SSM, COMPLETION_ST, FINAL_SSM), reviewer_id, decision, reason, created_at. SSM reviewer must belong to creator ST's team; COMPLETION_ST reviewer must equal creator_st_id.
- work_order_events: id, work_order_id, actor_id, action, details, created_at.
- photos: id, work_order_id, relative_path, uploaded_at.

Operational status enum: OPEN, PENDING, COMPLETED, CLOSED, CANCELLED. CANCELLED and CLOSED are terminal. Approval stage and operational status are separate. Initial SSM approval OPEN → PENDING; technician completion PENDING → COMPLETED; creator ST approval keeps COMPLETED; final SSM approval COMPLETED → CLOSED. Any review rejection → CANCELLED with mandatory reason.
Use SQLite partial unique index on equipment_id WHERE operational_status NOT IN ('CLOSED','CANCELLED') to ensure one active order per equipment, including COMPLETED awaiting final approval. Use transactions for approval and event writes. Exact field names are draft until contract review.