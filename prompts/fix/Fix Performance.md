I need you to thoroughly review and fix all real performance bottlenecks in the project.

Scope:

- Review the report at `result\result-Performance.md`
  - Verify each performance defect (slow queries, high latency, low throughput) is reproducible under realistic load
  - Remove any false positives (e.g., acceptable p95 under expected concurrency)
  - For confirmed issues, implement proper production-grade fixes (indexes, caching, query optimization, connection pooling) — no temporary patches
- Ensure that after fixes, p95 latency and throughput meet the defined SLOs (if any, otherwise reasonable defaults)

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No degraded performance compared to baseline
  - No new errors or timeouts under load testing (or manual simulation)
  - Resource usage (CPU, memory, DB connections) stays within limits
- Fix all issues until the application runs efficiently under expected load

Standards:

- Follow the Source of Truth: `wiki\00-source-of-truth.md`
- If code deviates from the Wiki → refactor code to match the Wiki
- If code is better than the Wiki → update the Wiki first, then align the code accordingly

Execution rules:

- Do not skip any performance defect
- Do not assume the report is correct — verify with profiling, `EXPLAIN ANALYZE`, or load test scripts
- Do not ignore inconsistencies between code and documentation
- Ensure the system is performant, scalable within its design limits, and aligned with defined standards
