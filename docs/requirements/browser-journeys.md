# Browser journeys and accessibility

This release baseline retains the supplied requirement identifiers and behavioral boundaries. Detailed implementation choices are linked from the component design index.

## Browser workspace

Screens: stream list and creation; synthetic append; event explorer with offset navigation; segment/integrity view; consumer-group lag and ownership; bounded replay controls; projection comparison; retention preview/blockers; audit and readiness.

Always distinguish highWater from consumer nextOffset and completed replay from partial output. Show retained interval, epoch, pending action and unsupported reducer/schema errors. Explain retention consequences before group retirement or deletion. Refresh does not reset progress. Two tabs use expected revisions and show conflicts. Provide keyboard operation, text status labels and tables equivalent to charts. Bundle all assets locally.
