# Worked examples

These are small fictional request and protocol examples, not the canonical stress fixtures or recorded test results. Each JSON file has a corresponding schema. Digest-bearing decision inputs must use actual canonical digests fetched from the current reviewed state; no invented digest is supplied here.

| Example | Schema | What it demonstrates |
| --- | --- | --- |
| [device-lifecycle-append.json](device-lifecycle-append.json) | [append-request](../schemas/append-request.schema.json) | An ordered four-event batch. On an empty stream the accepted interval is [0,4); retry the producer key to recover that same interval. |
| [checkpoint-request.json](checkpoint-request.json) | [checkpoint-request](../schemas/checkpoint-request.schema.json) | Commit only after local projection commit and only through the contiguous delivered range. Epoch 4 is illustrative, not a live token. |
| [bounded-replay.json](bounded-replay.json) | [replay-request](../schemas/replay-request.schema.json) | Use fresh identities and a captured end offset. Later appends do not expand this replay. |
| [stream-request.json](stream-request.json) | [stream-request](../schemas/stream-request.schema.json) | Illustrative capacity/age choices, not measured targets. Durable group and replay pins still override desired pruning. |
| [group-request.json](group-request.json) | [group-request](../schemas/group-request.schema.json) | Requires offset 0 to remain retained. An out-of-range start fails explicitly. |

Submit mutations with a stable Idempotency-Key and the appropriate fictional role/registered process. Retry the same intent with the same key and body after an uncertain response. A new edited intent needs a new key. An accepted response is required before the browser displays success.

Actual generated large fixtures, independent expected-output hashes and crash evidence are future implementation deliverables. These files claim no measured results.
