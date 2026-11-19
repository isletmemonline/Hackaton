# Workflows
## Operational status and approvals
Operational statuses: OPEN, PENDING, COMPLETED, CLOSED, CANCELLED. Only CLOSED and CANCELLED are terminal. Approval stages (INITIAL_SSM, COMPLETION_ST, FINAL_SSM) are separate from operational status. All nonterminal orders count as active for equipment uniqueness.

1. Creator ST creates immutable INS/REP/REM work order for exactly one equipment belonging to a customer assigned to that ST; sole technician auto-assigned; status OPEN.
2. Before initial SSM approval, only creator ST may cancel. An SSM authorized for the creator ST's team approves initial review (OPEN → PENDING) or rejects (OPEN → CANCELLED, mandatory reason).
3. Assigned technician may work only after initial approval; completes with nonblank explanation and at least one photo (PENDING → COMPLETED), pending creator ST completion review.
4. Only creator ST approves completion (COMPLETED stays COMPLETED, next review FINAL_SSM) or rejects (COMPLETED → CANCELLED, mandatory reason).
5. An SSM authorized for creator ST's team gives final approval (COMPLETED → CLOSED) after ST approval, or rejects (COMPLETED → CANCELLED, mandatory reason).
6. All approvals and actions appear in chronological detail timeline, with actor, time, outcome, reason. Technician evidence immutable.
7. A completed order awaiting review remains active for one-order-per-equipment enforcement.
8. Backend verifies identity, role, team/customer/equipment relationship and approval stage for each action. Reject skipping steps or duplicate conflicting decisions.