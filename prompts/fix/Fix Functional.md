I need you to thoroughly review and fix all real functional defects in the project.

Scope:

- Review the report at `system-prompt-qa-suite\prompts\result\result-Functional.md`
- Write the fix report to `system-prompt-qa-suite\prompts\report\report-Functional.md` documenting all applied fixes
  - Verify each functional defect is reproducible and violates business requirements
  - Remove any false positives (e.g., expected behavior mistaken as bug)
  - For confirmed issues, implement proper production-grade fixes (no temporary patches)
- Ensure all workflows (e.g., order → payment → fulfillment) behave correctly under valid and invalid inputs
- Verify error handling returns appropriate HTTP status codes and user-friendly messages (no stack traces)

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No business logic failures (e.g., state transitions, idempotency)
  - No broken API contracts (request/response validation)
  - No workflow blockages or unintended side effects
- Fix all issues until the application passes all functional acceptance criteria

Standards:

- Follow the Source of Truth: `system-prompt-qa-suite\wiki\00-source-of-truth.md`
- If code deviates from the Wiki → refactor code to match the Wiki
- If code is better than the Wiki → update the Wiki first, then align the code accordingly

Execution rules:

- Do not skip any functional defect
- Do not assume the report is correct — verify everything with actual requests and database state
- Do not ignore inconsistencies between code and documentation
- Ensure the system is fully functional, meets business requirements, and is aligned with defined standards
