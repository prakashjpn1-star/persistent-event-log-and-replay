# Data model and integrity contract

The data model separates authoritative state, in-progress work, immutable outcomes and audit history. Constraints preserve identity and prevent conflicting records.

## Logical data model

| Entity | Required data and constraints |
| --- | --- |
| Stream | Nonnull ID/generation/name, revision, admission state, low/high watermarks and policy. |
| Event | Unique stream/offset, immutable canonical payload/type/version/digest, producer identity and receipt reference. |
| AppendReceipt | Unique producer/operation/key, request digest and accepted offset range; retained across record pruning. |
| Segment | Stream, ordered ID, start/end-exclusive offsets, state, record/byte counts and sealed digest. |
| Group | Stream generation, unique name, nextOffset, owner instance/epoch, lease and state. |
| CheckpointReceipt | Scoped key/request digest and immutable committed progress response. |
| Replay | Stream/range, fresh group/projection identity, protected start, revision, status and result digest. |
| Projection metadata | Reducer version, source generation, nextOffset, output tables and applied identities in one consumer store. |
| Audit event | Unique stream sequence, actor/action, bounded details and timestamp. |
| Migration | Supported schema version, checksum and applied time. |

Constrain nonnull identities, nonnegative offsets, lowWater <= highWater, segment intervals, supported state values and referential integrity. Preserve records required to explain receipts after retention. Application guards enforce state-machine legality; storage checks protect structural invariants. Indexes and data deletion must be tested against realistic retained volumes.
