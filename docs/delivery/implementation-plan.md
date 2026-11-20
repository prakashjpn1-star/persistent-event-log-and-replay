# Delivery milestones and definition of done

This release baseline retains the supplied requirement identifiers and behavioral boundaries. Detailed implementation choices are linked from the component design index.

## Milestones

| Milestone | Checkable outcome |
| --- | --- |
| 1 | Stack/store verified; local UI, schema readiness and strict request contracts work. |
| 2 | Immutable stream creation and atomic append/receipts pass rollback and lost-response tests. |
| 3 | Ordered reads, logical segments, sealing and digests work at boundaries. |
| 4 | Groups, ownership leases and checkpoint fencing pass contention. |
| 5 | Deterministic projections recover across output/checkpoint crash windows. |
| 6 | Independent bounded replay, pause/resume and comparison work end to end. |
| 7 | Safe retention, lag diagnostics and cap backpressure pass race tests. |
| 8 | Browser investigation, authorization and error states are complete. |
| 9 | Full failure matrix, offline demo, soak, performance and restore evidence complete. |

Each milestone can span several sessions. Maintain commits, continuation notes, unresolved defects and the next concrete task. The project remains substantial even if the storage product supplies primitive durability; consumer semantics and coordinated recovery must still be implemented and proved.

## Deliverables and done

Deliver source/licence notices, migrations, local launch instructions, append/read/checkpoint contracts, fixture generator with independently calculated outputs, browser application, reducer examples, fault harness, acceptance report, measurements and backup/recovery guide. Every mandatory acceptance case and repetition gate must pass; unresolved correctness defects block release. Neither a schema validator nor an attractive event table proves runtime recovery.

Open choices: language/framework, one canonical store, projection store, serialization/digests, exact segment/retention bounds and package versions. Any later file-based storage or partitioning requires revised guarantees and its own failure tests.
