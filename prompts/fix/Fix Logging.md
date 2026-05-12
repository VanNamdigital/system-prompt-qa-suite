I need you to thoroughly review and fix all real logging and observability gaps in the project.

Scope:

- Review the report at `system-prompt-qa-suite\prompts\result\result-Logging.md`
- Write the fix report to `system-prompt-qa-suite\prompts\report\report-Logging.md` documenting all applied fixes
  - Verify each observability defect (missing structured logs, no correlation IDs, absent metrics, untraceable dependencies, noisy alerts) is real and impacts incident response time
  - Remove any false positives (e.g., low-severity missing metric that is not critical)
  - For confirmed issues, implement proper production-grade fixes (add JSON logging with request_id, instrument HTTP/RPC calls with histograms, propagate trace context, create actionable alerts) — no temporary patches
- Ensure that an on-call engineer can answer "what broke, where, why" within 30 seconds using available telemetry

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No runtime errors due to logging changes
  - Logs are structured and contain necessary fields (timestamp, severity, request_id)
  - Metrics endpoints (e.g., /metrics) expose RED/USE data
  - Traces (if implemented) correctly propagate across services
- Fix all issues until the system is fully observable

Standards:

- Follow the Source of Truth: `system-prompt-qa-suite\wiki\00-source-of-truth.md`
- If code deviates from the Wiki → refactor to match the Wiki
- If code is better (e.g., more detailed logging) → update the Wiki first

Execution rules:

- Do not skip any observability defect
- Do not assume the report is correct — verify by triggering a fake error and checking logs/metrics/traces
- Do not ignore inconsistencies between code and documentation
- Ensure the system is observable, debuggable, and aligned with defined standards
