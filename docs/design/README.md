# Low-level component designs

Every component follows the same [transaction rules](transactions.md), [API errors](../contracts/api.md) and [data constraints](../contracts/data-model.md).

- [Atomic append and producer receipts](append.md)
- [Logical segments and integrity](segments.md)
- [Consistent offset reads](reads.md)
- [Group ownership and checkpoint fencing](groups.md)
- [Deterministic projections and recovery](projections.md)
- [Replay and safe prefix retention](replay-retention.md)
- [State machines](state-machines.md)
- [Browser and control API](browser-and-api.md)
- [Observability](observability.md)

Each design includes an algorithm, diagram, atomicity/recovery behavior and applicable source contracts. The [test harness](../testing/harness-and-fixtures.md) defines evidence required to prove the design.

- [Storage and entity relationships](storage.md)
