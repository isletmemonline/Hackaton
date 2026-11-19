# Work-order flow skill
1. Read docs/WORKFLOWS.md and DECISIONS.md.
2. Verify actor and team/customer scope.
3. Verify current approval step and operational status.
4. Execute transition atomically, write approval decision and immutable event.
5. Check idempotency and all cancellation paths.
6. Test both success and unauthorized/repeated requests.