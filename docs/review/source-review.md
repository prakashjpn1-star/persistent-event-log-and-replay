# Source review and resolved design questions

The main design risks concern the boundary between a committed outcome and a lost response. The following decisions address those risks; runtime acceptance evidence is still pending.

| Question or risk | Design response |
| --- | --- |
| Server checkpoint ahead of the projection would silently lose derived state. | The consumer commits output and its local checkpoint before requesting server progress. Server-ahead-local is an explicit consistency error. |
| A paused durable group still blocks retention and may cause admission backpressure. | Paused groups retain their checkpoint pin. Retention diagnostics identify blockers and admission returns backpressure at the hard cap. |
| Retained-away offset receipts must remain replayable without re-appending. | Append receipts outlive pruned records and return the original accepted interval without inserting new events. |
| A sealed digest is an application integrity check, not a claim of storage-engine power-loss recovery. | Segment verification checks retained application records; corruption produces an explicit affected-range failure and preserves evidence. |
| Dependency compatibility | Python and SQLite are the reference stack. Exact framework/library versions depend on the permitted environment inventory. |
| Expected hashes not yet generated | Require deterministic generator and independent oracle; keep actual result fields unfilled. |
| Durability confused with exactly-once effects | State acceptance, delivery and local-effect guarantees separately. |
| Generated evidence overwritten | Keep result reports separate; no build script regenerates executed acceptance outcomes. |

New design choices are labeled proposed: explicit SQLite writer serialization, canonical JSON library, one-second browser polling and conservative recovery. Open items include exact versions, configured hard caps, workload measurements and immutable-field migration enforcement.

The original requirements remain archived for traceability. Programme eligibility and participant authorship are assessed separately from these technical design decisions.
