# Requirement traceability

Every source section is covered below, including requirements stated in prose. Every numbered acceptance case is retained verbatim in the acceptance plan. Coverage describes intended verification, not test success.

| Source section | Requirement/design coverage | Verification |
| --- | --- | --- |
| 1 Purpose and release outcome | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 2 Users and concrete example | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 3 Desktop constraints and implementation freedom | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 4 Scope | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 5 Terms and delivery semantics | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 6 Reference journey | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 7 Stream and event requirements | [Contract](../requirements/software.md); [Design](../design/append.md) | Acceptance plan and milestone gate |
| 8 Atomic append and receipts | [Contract](../requirements/software.md); [Design](../design/append.md) | Acceptance plan and milestone gate |
| 9 Logical segments integrity and indexes | [Contract](../requirements/software.md); [Design](../design/segments.md) | Acceptance plan and milestone gate |
| 10 Read contract | [Contract](../requirements/software.md); [Design](../design/reads.md) | Acceptance plan and milestone gate |
| 11 Consumer groups and ownership | [Contract](../requirements/software.md); [Design](../design/groups.md) | Acceptance plan and milestone gate |
| 12 Projection consumer and replay correctness | [Contract](../requirements/software.md); [Design](../design/projections.md) | Acceptance plan and milestone gate |
| 13 Retention and backpressure | [Contract](../requirements/software.md); [Design](../design/replay-retention.md) | Acceptance plan and milestone gate |
| 14 Transaction and recovery protocol | [Contract](../requirements/software.md); [Design](../design/segments.md) | Acceptance plan and milestone gate |
| 15 Logical data model | [Contract](../contracts/data-model.md) | Acceptance plan and milestone gate |
| 16 API requirements | [Contract](../contracts/api.md) | Acceptance plan and milestone gate |
| 17 Browser workspace | [Contract](../requirements/browser-journeys.md) | Acceptance plan and milestone gate |
| 18 Access observability and operations | [Contract](../security/access-and-threats.md) | Acceptance plan and milestone gate |
| 19 Provisional targets | [Contract](../operations/runbook.md) | Acceptance plan and milestone gate |
| 20 Acceptance catalog | [Contract](../testing/acceptance-plan.md) | Acceptance plan and milestone gate |
| 21 Test methodology | [Contract](../testing/acceptance-plan.md) | Acceptance plan and milestone gate |
| 22 Milestones | [Contract](../delivery/implementation-plan.md) | Acceptance plan and milestone gate |
| 23 Deliverables and done | [Contract](../delivery/implementation-plan.md) | Acceptance plan and milestone gate |
| 24 Primary references | [Contract](../architecture/references.md) | Acceptance plan and milestone gate |

## Numbered functional requirements

| Source ID | Required behavior | Design |
| --- | --- | --- |
| STR-01 | Create a named stream with immutable ID/generation, capacity and versioned retention policy. Reject duplicate active names. | [Atomic append and producer receipts](../design/append.md) |
| STR-02 | Events contain producer identity, producer request key, type/schema version, bounded payload, accepted time and content digest. | [Atomic append and producer receipts](../design/append.md) |
| STR-03 | Append assigns consecutive offsets in transaction commit order. Rollback exposes neither record nor advanced highWater. | [Atomic append and producer receipts](../design/append.md) |
| STR-04 | Events cannot be edited in place. Corrections are new events with explicit reference to prior events. | [Atomic append and producer receipts](../design/append.md) |
| STR-05 | Pause admission rejects new appends while retained reads and consumer settlement continue. | [Atomic append and producer receipts](../design/append.md) |
| STR-06 | First release has no automatic stream deletion/recreation. A name must not silently reset old offset/checkpoint identities. | [Atomic append and producer receipts](../design/append.md) |
| APP-01 | One append operation targets one stream and contains 1 to 100 events, subject to a total request bound. The batch is all-or-nothing. | [Atomic append and producer receipts](../design/append.md) |
| APP-02 | The producer supplies an operation key. Same key and canonical batch returns the same offset range; changed content conflicts. | [Atomic append and producer receipts](../design/append.md) |
| APP-03 | Record inserts, segment membership, next offset, accepted event metadata and receipt commit in one transaction. | [Atomic append and producer receipts](../design/append.md) |
| APP-04 | Confirmation is returned only after the store reports durable commit using the selected durability configuration. | [Atomic append and producer receipts](../design/append.md) |
| APP-05 | Producer receipts survive segment retention so an old key cannot create a second accepted batch. Replays may reference offsets already retained away. | [Atomic append and producer receipts](../design/append.md) |
| APP-06 | Capacity failure rejects the batch. If retention is blocked, return backpressure rather than deleting protected records. | [Atomic append and producer receipts](../design/append.md) |
| APP-07 | Validate serialization, sizes and schemas before acquiring a write transaction, then recheck mutable stream state inside it. | [Atomic append and producer receipts](../design/append.md) |
| CON-01 | Create a named group bound to one stream generation and initial nextOffset within the retained range. | [Group ownership and checkpoint fencing](../design/groups.md) |
| CON-02 | Only one local consumer instance owns a group at a time. Acquire ownership atomically and increment owner epoch. | [Group ownership and checkpoint fencing](../design/groups.md) |
| CON-03 | Default ownership lease is 15 seconds with heartbeat every 5 seconds. Expired ownership cannot be revived. | [Group ownership and checkpoint fencing](../design/groups.md) |
| CON-04 | Fetch/commit carries group ID, instance ID and epoch; stale owners cannot advance progress. | [Group ownership and checkpoint fencing](../design/groups.md) |
| CON-05 | Track the current owner's contiguous delivered range. A commit cannot exceed deliveredEndExclusive or move the checkpoint backward. | [Group ownership and checkpoint fencing](../design/groups.md) |
| CON-06 | A committed checkpoint means the client asserts processing through that range. The server cannot prove arbitrary consumer effects occurred. | [Group ownership and checkpoint fencing](../design/groups.md) |
| CON-07 | Idle or stopped durable groups continue to pin retention until explicitly retired. Show that cost in the UI. | [Group ownership and checkpoint fencing](../design/groups.md) |
| CON-08 | Group retirement is explicit, revision-checked and irreversible in this release; it releases retention protection and revokes ownership. | [Group ownership and checkpoint fencing](../design/groups.md) |
| CON-09 | Operator pause/resume uses expected group revision. Pause revokes the current owner epoch and blocks new acquisition/fetch; its checkpoint still protects retention. Resume permits a fresh epoch without resetting progress. | [Group ownership and checkpoint fencing](../design/groups.md) |

## Acceptance coverage

| Source case | Durable acceptance oracle | Status |
| --- | --- | --- |
| EL-AC-01 | Create streams; reject duplicate identities and invalid policy/size/schema inputs. | NOT RUN |
| EL-AC-02 | Concurrent append assigns one consecutive per-stream committed sequence with valid receipts. | NOT RUN |
| EL-AC-03 | Whole-batch rollback leaves neither records nor advanced highWater. | NOT RUN |
| EL-AC-04 | Lost append response and same-key retry return original offsets; changed batch conflicts. | NOT RUN |
| EL-AC-05 | Restart after confirmation preserves accepted records, segment bounds and receipts. | NOT RUN |
| EL-AC-06 | Seal/rotate segment and verify ordered content digest; altered test content raises explicit integrity failure. | NOT RUN |
| EL-AC-07 | Two groups progress independently; pause/slowdown in one does not change the other's checkpoint. | NOT RUN |
| EL-AC-08 | Competing group owners yield one current epoch; old heartbeat/commit is rejected. | NOT RUN |
| EL-AC-09 | Kill consumer after fetch before output; restart reprocesses safely from durable progress. | NOT RUN |
| EL-AC-10 | Kill after projection commit before broker checkpoint; replayed records do not duplicate local effects. | NOT RUN |
| EL-AC-11 | Lose checkpoint response; retry receipt preserves one monotonic commit. | NOT RUN |
| EL-AC-12 | Checkpoint beyond delivered range or backward movement is rejected. | NOT RUN |
| EL-AC-13 | Fixed replay end ignores subsequent appends and matches independent reference hash. | NOT RUN |
| EL-AC-14 | Replay pause/restart/cancel preserves committed output and recorded position. | NOT RUN |
| EL-AC-15 | Retention removes only safe sealed prefix; blocked group/replay is reported. | NOT RUN |
| EL-AC-16 | Read/retention races return consistent records and bounds, never silent skipping. | NOT RUN |
| EL-AC-17 | Read below lowWater returns explicit range failure; old append key remains deduplicated after pruning. | NOT RUN |
| EL-AC-18 | At hard storage cap, reject new append rather than delete protected input. | NOT RUN |
| EL-AC-19 | Storage outage stops false confirmation/progress; recovery resolves uncertain requests. | NOT RUN |
| EL-AC-20 | Full 20,000-event demo yields matching device-state and summary projections after crash injection. | NOT RUN |
| EL-AC-21 | Unknown reducer/schema halts explicitly; invalid business fixture is diagnosable without corrupting log ordering. | NOT RUN |
| EL-AC-22 | Browser reload/two tabs, pagination boundaries and authorization checks behave consistently. | NOT RUN |
| EL-AC-23 | Consistent restore preserves retained events, receipt identities and safe projection recovery. | NOT RUN |
| EL-AC-24 | Complete append/consume/replay/inspect journey runs with external connectivity disabled. | NOT RUN |
