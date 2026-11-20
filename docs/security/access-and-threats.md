# Local access and threat model

This release baseline retains the supplied requirement identifiers and behavioral boundaries. Detailed implementation choices are linked from the component design index.

## Access observability and operations

Loopback binding, strict same-origin mutation checks and disposable fictional operator/producer/consumer roles are required. No real account sign-in, real secrets, external URLs or arbitrary input paths. Allowlisted fixtures/reducers only; fault controls are disabled in normal mode. Bound JSON nesting, payloads, logs and inspection responses.

Metrics: append latency/rate, high/low watermarks, retained bytes, group lag, owner expiry, stale-epoch rejection, replay progress, retention blockers and integrity failures. Audit committed append, ownership, checkpoint, replay and retention transitions in transaction order. Avoid an audit record per heartbeat when it would create unbounded noise.

Use explicit checksummed migrations and consistent store snapshots including receipts and retention metadata. For end-to-end projection restore, stop relevant processes and take a coordinated cold snapshot of canonical and projection stores, or document a tested independent-recovery protocol. Do not assume separately timed live copies are consistent. Restore with fresh process identities and original checkpoints. Keep actual evidence reports separate from regenerable templates.
