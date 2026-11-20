# Software requirements specification

Offsets are ordered within one stream generation. Canonical log acceptance and a consumer’s local projection are separate commits. Redelivery is expected; projection writes and their local checkpoint must be atomic.

## Stream and event requirements

| ID | Requirement |
| --- | --- |
| STR-01 | Create a named stream with immutable ID/generation, capacity and versioned retention policy. Reject duplicate active names. |
| STR-02 | Events contain producer identity, producer request key, type/schema version, bounded payload, accepted time and content digest. |
| STR-03 | Append assigns consecutive offsets in transaction commit order. Rollback exposes neither record nor advanced highWater. |
| STR-04 | Events cannot be edited in place. Corrections are new events with explicit reference to prior events. |
| STR-05 | Pause admission rejects new appends while retained reads and consumer settlement continue. |
| STR-06 | First release has no automatic stream deletion/recreation. A name must not silently reset old offset/checkpoint identities. |

Each accepted stream declares supported payload schemas; validate event type/version at append. Old events retain their original schema. New schema versions require registered reducers compatible with those versions; unsupported events stop a projection explicitly instead of silently skipping.

## Atomic append and receipts

| ID | Requirement |
| --- | --- |
| APP-01 | One append operation targets one stream and contains 1 to 100 events, subject to a total request bound. The batch is all-or-nothing. |
| APP-02 | The producer supplies an operation key. Same key and canonical batch returns the same offset range; changed content conflicts. |
| APP-03 | Record inserts, segment membership, next offset, accepted event metadata and receipt commit in one transaction. |
| APP-04 | Confirmation is returned only after the store reports durable commit using the selected durability configuration. |
| APP-05 | Producer receipts survive segment retention so an old key cannot create a second accepted batch. Replays may reference offsets already retained away. |
| APP-06 | Capacity failure rejects the batch. If retention is blocked, return backpressure rather than deleting protected records. |
| APP-07 | Validate serialization, sizes and schemas before acquiring a write transaction, then recheck mutable stream state inside it. |

Default payload maximum is 64 KiB, batch maximum 1 MiB encoded, metadata maximum 8 KiB per event. Reject duplicate JSON keys, nonfinite numbers and ambiguous encodings. Define canonical bytes/digest algorithm once. The request digest includes stream generation, event order, types, versions and payloads. Store individual event digests too.

## Logical segments integrity and indexes

Assign events to logical segments with configurable record/byte thresholds, initially 1,000 records or 8 MiB. A segment owns a contiguous offset interval. Seal before starting its successor; sealing records the immutable interval, count and digest over ordered event identity and canonical content. Persist an empty current segment only if the format explicitly supports it.

Sealed segment digest verification must detect altered/reordered/missing content. This verifies retained application records, not the underlying disk's correctness. An OPEN segment has individually verified records and can be sealed during recovery or clean shutdown. Transaction rollback prevents partially accepted records from becoming visible.

Index stream/offset, event key/type where required, group checkpoint, owner expiry and segment bounds. The authoritative data is committed records; rebuild optional indexes from that data in a controlled operation. Never silently discard a damaged sealed segment. Mark the stream unavailable for affected reads, preserve evidence and offer restore instructions.

## Read contract

Reads specify stream ID/generation, start offset, maximum events and maximum response bytes. Return consecutive records in offset order with nextOffset, lowWater, highWater and relevant digests. Default page is 100 events, maximum 1,000, bounded by 1 MiB response bytes.

At highWater return an explicit empty result with unchanged nextOffset. Requests below lowWater return OFFSET_OUT_OF_RANGE with current bounds; do not silently fast-forward. Requests above highWater are rejected. A page is read from one consistent snapshot and cannot combine a deleted segment boundary with stale metadata.

A read need not commit group progress. Bounded replay reads also carry endExclusive, captured at replay creation, and never consume later appends. Polling rather than a streaming socket is acceptable. Business-event timestamps can be searched, but only an explicit resulting offset controls replay.

## Consumer groups and ownership

| ID | Requirement |
| --- | --- |
| CON-01 | Create a named group bound to one stream generation and initial nextOffset within the retained range. |
| CON-02 | Only one local consumer instance owns a group at a time. Acquire ownership atomically and increment owner epoch. |
| CON-03 | Default ownership lease is 15 seconds with heartbeat every 5 seconds. Expired ownership cannot be revived. |
| CON-04 | Fetch/commit carries group ID, instance ID and epoch; stale owners cannot advance progress. |
| CON-05 | Track the current owner's contiguous delivered range. A commit cannot exceed deliveredEndExclusive or move the checkpoint backward. |
| CON-06 | A committed checkpoint means the client asserts processing through that range. The server cannot prove arbitrary consumer effects occurred. |
| CON-07 | Idle or stopped durable groups continue to pin retention until explicitly retired. Show that cost in the UI. |
| CON-08 | Group retirement is explicit, revision-checked and irreversible in this release; it releases retention protection and revokes ownership. |
| CON-09 | Operator pause/resume uses expected group revision. Pause revokes the current owner epoch and blocks new acquisition/fetch; its checkpoint still protects retention. Resume permits a fresh epoch without resetting progress. |

Transport retries preserve operation keys. A matching committed checkpoint receipt can be replayed after lease expiry; that proves past progress and grants no current ownership. Empty fetches and heartbeats have documented key lifetimes. Counted checkpoints and lease updates come from committed state, never process memory alone.

Group control state is ACTIVE, PAUSED or RETIRED. If a consumer finishes a local projection batch after group pause, its now-stale server checkpoint commit is rejected. On resume, normal idempotent projection recovery reconciles that already committed local output before advancing server progress.

## Projection consumer and replay correctness

The built-in reducer reads batches sequentially and commits all derived writes, an applied-event identity guard and its nextOffset in one local projection transaction. If a batch fails, roll it back completely. Use deterministic integer/string operations and stable serialization for expected hashes; avoid wall-clock or random values in canonical results.

Only after local projection commit does the consumer ask the service to advance the group checkpoint. If that response is lost, retry the same key. If server progress lags the local projection after restart, reread from the server checkpoint and recognize already applied offsets without repeating effects, then advance safely. Server progress ahead of the local projection is a consistency error requiring investigation, not permission to skip data.

Projection storage may be separate from broker storage; do not claim one atomic transaction spans them. Independent replay uses a fresh group and projection ID, preserving the original group's checkpoint. Replay state is CREATED, RUNNING, PAUSED, SUCCEEDED, FAILED or CANCELLED. Cancellation stops future batches; already committed projection batches remain inspectable. Whole-stream record replay is not arbitrary program execution.

## Retention and backpressure

Retention deletes only a contiguous prefix of SEALED segments. The proposed cutoff must satisfy age/size policy and be no greater than every active durable group's checkpoint and every active replay's protected start. Never delete the OPEN segment or skip a protected segment to delete later ones.

Baseline canonical records and metadata are in one transactional store, so prefix deletion and lowWater advancement commit together. Producer receipts and audit remain retained. Reads and retention use snapshot/transaction rules preventing inconsistent bounds. Custom file deletion would need a separate crash-safe deletion protocol and is deferred.

Paused or failed consumers can block deletion. Show the blocking group and storage pressure. At the configured hard cap, reject appends until safe capacity becomes available; do not discard unprocessed data. Registering a new group never resurrects already deleted data. First release has no snapshots for replay from a pruned prefix: clients need a retained baseline or must receive a clear unavailable-range error.

## Transaction and recovery protocol

Document isolation, lock ordering and linearization for append, owner acquisition, checkpoint commit, retention and replay control. Recheck stream/group revision, owner epoch and durable time within the accepting transaction. Shared limits and metadata must be protected as well as the selected row.

On startup verify schema compatibility, consistency of highWater/segments, due owner expiry and replay status. Do not reset checkpoints or grant old ownership fresh leases. Several reconciliation loops must be safe through conditional updates and unique constraints.

When storage is unavailable, stop confirmations and progress commits. Handle uncertain commit results through durable keyed receipts. Graceful producers finish current append; graceful consumers finish bounded batches and release ownership. Hard stops recover through lease expiry and retained checkpoints. A recovery scan verifies segment boundaries/digests without rewriting history to hide damage.
