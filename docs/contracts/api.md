# API contract

This release baseline retains the supplied requirement identifiers and behavioral boundaries. Detailed implementation choices are linked from the component design index.

## API requirements

| Method and path | Intended behaviour |
| --- | --- |
| GET /api/health and /api/ready | Separate liveness from storage/schema/integrity readiness. |
| POST /api/streams | Create stream and policies. |
| POST /api/streams/{id}/append | Keyed atomic batch append. |
| GET /api/streams/{id}/events | Bounded offset-range read. |
| GET /api/streams/{id}/segments | Segment bounds, sizes and verification state. |
| POST /api/streams/{id}/pause, /resume | Control new append admission. |
| POST /api/groups | Create durable group with explicit starting offset. |
| POST /api/groups/{id}/pause, /resume | Revision-checked control without resetting the checkpoint or releasing retention protection. |
| POST /internal/groups/{id}/acquire, /heartbeat, /release | Fenced group ownership. |
| GET /internal/groups/{id}/events | Current-owner bounded fetch. |
| POST /internal/groups/{id}/checkpoint | Keyed monotonic delivered-range commit. |
| POST /api/groups/{id}/retire | Explicitly release durable retention protection. |
| POST /api/replays | Create fresh bounded replay/projection. |
| POST /api/replays/{id}/pause, /resume, /cancel | Revision-checked replay controls. |
| POST /api/streams/{id}/retention/preview, /apply | Preview safe deletion and apply revalidated cutoff. |
| GET /api/projections/{id}, /api/audit and /api/metrics | Inspect derived output and operational evidence. |

Append example: `{"events":[{"type":"device-created","schemaVersion":1,"payload":{"deviceId":"synthetic-001","label":"sample"}}]}` with producer identity and operation key outside event payload. Checkpoint example: `{"instanceId":"local-consumer-1","epoch":4,"nextOffset":100}`.

All mutations define scoped idempotency and expected-revision rules. Use 400 malformed request, 422 contract error, 403 denied identity, 404 absent resource, 409 stale epoch/key/revision, 413 size limit, 416 unavailable offset range and 503 storage unavailable. Return structured error code/message/correlation ID and current bounds where safe. Paginate administration lists with stable cursors, default 50/maximum 200.
