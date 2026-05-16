I need you to thoroughly review and fix all real API contract and integration issues in the project.

Scope:

- Review the report at `system-prompt-qa-suite\prompts\result\result-API Contract and Integration.md`
- Write the fix report to `system-prompt-qa-suite\prompts\report\report-API Contract and Integration.md` documenting all applied fixes
  - Verify each API contract defect (spec-code drift, missing documentation, breaking changes, inconsistent error format, missing contract tests) is reproducible via spec comparison or integration test
  - Remove any false positives (e.g., internal-only endpoint not meant for public documentation)
  - For confirmed issues, implement proper production-grade fixes (align spec with implementation, add missing endpoints to spec, enforce consistent error schema, add consumer-driven contract tests, implement API versioning strategy) — no temporary patches
- Ensure all API contracts are formally defined, versioned, and verified between producers and consumers

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No spec validation errors (OpenAPI/Swagger/GraphQL schema passes validator)
  - Contract tests pass for all critical integration points
  - Error responses follow a consistent format with unique error codes
  - No broken integrations between services
  - API documentation renders correctly (if documentation portal exists)
- Fix all issues until APIs are contractually solid, documented, and integration-safe

Standards:

- Follow the Source of Truth: `wiki\00-source-of-truth.md`
- If API code deviates from the Wiki → refactor to match the Wiki
- If current API design is better → update the Wiki first

Execution rules:

- Do not skip any API contract defect
- Do not assume the report is correct — verify with actual HTTP responses against spec schemas
- Do not ignore inconsistencies between code and documentation
- Ensure the system has reliable, documented, and versioned APIs aligned with defined standards
