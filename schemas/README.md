# Schemas and reference storage model

These JSON Schemas use draft 2020-12 and describe proposed version 1 contracts. They complement the [API contract](../docs/contracts/api.md); the runtime is not implemented yet. The field limits newly made concrete here are proposal defaults, not measured limits or silent changes to the source scope.

| Schema | Purpose |
| --- | --- |
| [event.schema.json](event.schema.json) | Version 1 synthetic device event |
| [append-request.schema.json](append-request.schema.json) | Atomic ordered append request |
| [checkpoint-request.schema.json](checkpoint-request.schema.json) | Fenced group checkpoint commit |
| [replay-request.schema.json](replay-request.schema.json) | Independent bounded replay creation |
| [stream-request.schema.json](stream-request.schema.json) | Stream creation and retention policy |
| [group-request.schema.json](group-request.schema.json) | Durable consumer group creation |

[reference-schema.sql](reference-schema.sql) supplies relational keys, foreign keys, enum constraints and indexes. Sections labeled as separate stores belong in separate databases. This is a reference design, not an installed migration. Migration checksums, immutable-field triggers, role checks, byte limits and the complete transaction protocol remain implementation work.

JSON Schema checks structure, not database state. It cannot prove a current revision, consecutive feed/offset range, valid lease, safe retention cutoff or sufficient capacity. Enforce those rules in the accepting transaction. Encoded byte limits, duplicate-key rejection, Unicode/canonicalization and SHA-256 verification also require the parser/domain implementation.

Idempotency-Key, correlation IDs and demo identity normally travel in headers/session context and are not duplicated in these body schemas. Path IDs must agree with any body identity. Examples use fictional readable IDs; resolve names to registered IDs before actual submission.

Device event payloads define an illustrative version 1 reducer contract. Reading is a bounded integer; lifecycle legality is a reducer check. Replay start must be within retained bounds and endExclusive must be at least start and no greater than captured highWater.

No schema parser or SQL setup result is a runtime durability acceptance result.
