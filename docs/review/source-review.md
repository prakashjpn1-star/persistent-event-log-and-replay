# Source review and resolved design questions

The supplied scope is suitable for a substantial local reliability project. Its strongest feature is requiring durable evidence at the exact boundary where a process or response can be lost. This review expands that scope into an implementation design without changing its guarantees or claiming programme approval.

| Question or risk | Design response |
| --- | --- |
| Server checkpoint ahead of the projection would silently lose derived state. | Enforced by the corresponding component algorithm, atomic boundary and source acceptance case. |
| A paused durable group still blocks retention and may cause admission backpressure. | Enforced by the corresponding component algorithm, atomic boundary and source acceptance case. |
| Retained-away offset receipts must remain replayable without re-appending. | Enforced by the corresponding component algorithm, atomic boundary and source acceptance case. |
| A sealed digest is an application integrity check, not a claim of storage-engine power-loss recovery. | Enforced by the corresponding component algorithm, atomic boundary and source acceptance case. |
| Database/framework undecided | Proposed Python/SQLite architecture stays conditional on allowed package and desktop verification. |
| Expected hashes not yet generated | Require deterministic generator and independent oracle; keep actual result fields unfilled. |
| Durability confused with exactly-once effects | State acceptance, delivery and local-effect guarantees separately. |
| Generated evidence overwritten | Keep result reports separate; no build script regenerates executed acceptance outcomes. |

New design choices are labeled proposed: explicit SQLite writer serialization, canonical JSON library, one-second browser polling and conservative recovery. Open items include exact versions, configured hard caps, workload measurements and immutable-field migration enforcement.

The attached file is requirements evidence. Its evaluation/ownership directions are preserved as product/programme constraints, not treated as permission to access another workspace, submit forms or claim participant authorship. The user requested public GitHub documentation and MIT licensing; no implementation or programme submission is implied.
