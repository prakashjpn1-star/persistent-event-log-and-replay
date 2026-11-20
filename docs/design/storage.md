# Storage component and integrity design

The storage layer implements the atomic boundaries described by the domain services. Current state, authority, audit and receipts agree at commit. The entity relationships below are logical; cross-store associations are protocol checks rather than cross-database foreign keys.

```mermaid
erDiagram
    STREAM ||--o{ EVENT : orders
    STREAM ||--o{ SEGMENT : retains
    STREAM ||--o{ CONSUMER_GROUP : serves
    CONSUMER_GROUP ||--o{ CHECKPOINT_RECEIPT : commits
    REPLAY ||--|| CONSUMER_GROUP : isolates
    PROJECTION ||--o{ APPLIED_IDENTITY : guards
    PROJECTION ||--|| LOCAL_CHECKPOINT : advances
```

## Keys constraints and indexes

Canonical store: unique stream ID/generation and active name; unique (streamId,offset); unique producer/operation key receipt; unique stream/group name; valid nonnegative half-open segment bounds; lowWater <= highWater. Index events(streamId,offset), segments(streamId,startOffset), groups(streamId,state,nextOffset), ownership(leaseExpiry), and receipt scopes.

Projection store: unique (projectionId,streamGeneration,offset) applied identity with event digest and reducer version; local checkpoint advances in the same transaction as derived rows. Record unsupported schemas as explicit failures rather than skipping offsets.

Append receipts retain startOffset/endExclusive and request digest without cascading foreign keys to prunable events. Retention removes event/segment prefix and advances lowWater atomically, retaining receipts and audit. A group deliveredEndExclusive is scoped to its current owner epoch.

## Repository interfaces

Expose short transaction-scoped repository methods for receipt lookup, aggregate load, conditional transition, ordered audit allocation and outcome insert. Do not expose arbitrary table mutation to handlers or browser routes. API/coordinator code shares one domain transaction library. Return typed conflict/unavailable errors and preserve the original operation key on retry.

## Migration and durability rules

Use explicit supported schema versions, migration checksums, NOT NULL identities, foreign keys within each store, enumerated states and positive bounds. Immutable payload/receipt/history fields and terminal outcomes require service guards plus migration-level triggers. A nullable CHECK is insufficient for a mandatory identity. Do not delete records needed for receipt replay or lineage explanation.

Store JSON in the declared canonical format and bounded bytes separately from audit summaries. Persist database-evaluated UTC times and integer counters. Use a connection per bounded operation with foreign keys and durability settings verified. Avoid long read snapshots that retain excessive WAL pages. Unknown newer schemas fail readiness.

The complete entity dictionary is in [data-model contract](../contracts/data-model.md). Crash, unique-key, rollback and concurrent-writer tests remain implementation work; a documented schema is not evidence of those guarantees.
