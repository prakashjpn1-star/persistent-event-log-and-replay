# Acceptance and failure-test plan

This release baseline retains the supplied requirement identifiers and behavioral boundaries. Detailed implementation choices are linked from the component design index.

## Acceptance catalog

| ID | Scenario and required outcome |
| --- | --- |
| EL-AC-01 | Create streams; reject duplicate identities and invalid policy/size/schema inputs. |
| EL-AC-02 | Concurrent append assigns one consecutive per-stream committed sequence with valid receipts. |
| EL-AC-03 | Whole-batch rollback leaves neither records nor advanced highWater. |
| EL-AC-04 | Lost append response and same-key retry return original offsets; changed batch conflicts. |
| EL-AC-05 | Restart after confirmation preserves accepted records, segment bounds and receipts. |
| EL-AC-06 | Seal/rotate segment and verify ordered content digest; altered test content raises explicit integrity failure. |
| EL-AC-07 | Two groups progress independently; pause/slowdown in one does not change the other's checkpoint. |
| EL-AC-08 | Competing group owners yield one current epoch; old heartbeat/commit is rejected. |
| EL-AC-09 | Kill consumer after fetch before output; restart reprocesses safely from durable progress. |
| EL-AC-10 | Kill after projection commit before broker checkpoint; replayed records do not duplicate local effects. |
| EL-AC-11 | Lose checkpoint response; retry receipt preserves one monotonic commit. |
| EL-AC-12 | Checkpoint beyond delivered range or backward movement is rejected. |
| EL-AC-13 | Fixed replay end ignores subsequent appends and matches independent reference hash. |
| EL-AC-14 | Replay pause/restart/cancel preserves committed output and recorded position. |
| EL-AC-15 | Retention removes only safe sealed prefix; blocked group/replay is reported. |
| EL-AC-16 | Read/retention races return consistent records and bounds, never silent skipping. |
| EL-AC-17 | Read below lowWater returns explicit range failure; old append key remains deduplicated after pruning. |
| EL-AC-18 | At hard storage cap, reject new append rather than delete protected input. |
| EL-AC-19 | Storage outage stops false confirmation/progress; recovery resolves uncertain requests. |
| EL-AC-20 | Full 20,000-event demo yields matching device-state and summary projections after crash injection. |
| EL-AC-21 | Unknown reducer/schema halts explicitly; invalid business fixture is diagnosable without corrupting log ordering. |
| EL-AC-22 | Browser reload/two tabs, pagination boundaries and authorization checks behave consistently. |
| EL-AC-23 | Consistent restore preserves retained events, receipt identities and safe projection recovery. |
| EL-AC-24 | Complete append/consume/replay/inspect journey runs with external connectivity disabled. |

All cases start NOT RUN. Retain commit, environment, fixture seed, offsets, digests, process logs, receipts, invariant snapshots and exact pass/failure evidence.

## Test methodology

Unit tests cover reducer determinism, bounds, schema validation, digests and retention arithmetic. Integration tests exercise actual transactional append/checkpoint/retention. Concurrent-process tests use real local producers/consumers, not only mocked queues. Browser tests cover bounded replay, lag, retention warnings and stale views.

Deterministic barriers: before append commit; after append commit before response; after fetch; after projection commit before broker checkpoint; before owner expiry versus delayed checkpoint; and before retention commit versus read. Repeat crash windows at least 20 times and append/ownership/retention contention at least 100 rounds. Inspect invariants during transactions as well as after completion.

Invariants: immutable unique stream offsets; lowWater/highWater consistent with retained segments; checkpoints monotonic within a group; one authorized owner; no stale-epoch commit; no protected retention loss; output/checkpoint atomicity inside the projection store; same replay inputs and reducer produce same result. Do not treat redelivery as corruption or hide it behind an unsupported exactly-once claim.
