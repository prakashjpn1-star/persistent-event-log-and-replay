# Quality requirements and measurable targets

Correctness comes first: no fabricated confirmation, no stale authority mutation, and no loss of accepted history. Performance figures remain provisional until measured on the supplied desktop.


Initial measurable targets: 10 streams; 100,000 retained small events; 10 independent groups; four concurrent consumers across groups; 1,000-event/8 MiB logical segments; 100 small events per second over 60 seconds; p95 bounded reads below 500 ms; ownership expiry reconsidered within 5 seconds of deadline; UI convergence within 3 seconds; under 2 GiB service/store RAM excluding browser/build. A 30-minute soak must show no steadily growing leak or permanently stranded eligible consumer.

Record environment, payload sizes, store settings, group count and exact workload. These are proposals, not existing measurements. Retention pressure and slow reducers must be measured separately from raw append throughput.



## Shared release criteria

The complete journey works offline with loopback available, ordinary local processes and synthetic data. Browser state uses confirmed server revisions, text labels and keyboard-accessible equivalents. Inputs, pages, logs and artifact/record sizes are bounded before expensive work.

Measure p50/p95/max where relevant, record warm-up/sample count and retain raw failures. Every report includes build/commit, CPU/RAM/OS, store/interpreter/package versions, durability settings, worker counts, payload sizes, fixture seed and exact method. Do not silently relax a target or remove a slow sample.

Run crash windows at least 20 times, contention/authority races at least 100 rounds, and a bounded 30-minute soak. Report NOT RUN until evidence exists. Actual reports are separate from regenerable templates.
