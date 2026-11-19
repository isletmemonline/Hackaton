# Data model
- users: id, name, role (ST/SSM/TECHNICIAN), team_id, region_id.
- customers: id, name, assigned_st_id, region_id.
- equipment: id, customer_id, external_ref.
- work_orders: id, order_no, type (INS/REP/REM), equipment_id, customer_id, creator_st_id, assigned_technician_id, operational_status, explanation, created_at, completed_at, closed_at.
- approvals: id, work_order_id, stage (INITIAL_SSM, COMPLETION_ST, FINAL_SSM), reviewer_id, decision, reason, created_at.
- work_order_events: id, work_order_id, actor_id, action, details, created_at.
- photos: id, work_order_id, relative_path, uploaded_at.
Use SQLite partial unique index to ensure only one active order per equipment, including COMPLETED awaiting final approval. CANCELLED and CLOSED are terminal. Approval stage and operational status are separate. Exact field names are draft until contract review.