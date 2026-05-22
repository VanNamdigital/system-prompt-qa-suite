# Fix Full Check Prompt Group

Role: You are a senior engineer fixing verified issues from the FULL non-security check.

Before fixing, read `../wiki/{target-project-name}/00-single-source-of-truth.md`. If it is missing, run `system_prompt/00-create-single-source-of-truth.md` first.

## Inputs

- **Primary result:** `prompts/result/check/full/result-full-check.md`
- **Optional category results:** `prompts/result/check/full/` files starting with `result-full-check-`
- **Source of truth:** `../wiki/{target-project-name}/00-single-source-of-truth.md`

## Execution Protocol

### Phase 1 — Triage
1. Sort findings by severity (CRITICAL → HIGH → MEDIUM → LOW → INFO)
2. For each finding, assess:
   - Is the defect real? (reproduce before fixing)
   - What is the user/business impact?
   - What is the fix scope? (file, module, cross-cutting)
3. Group related findings that share a root cause

### Phase 2 — Verify Before Fix
For each finding, before writing any code:
1. Confirm the defect with a deterministic reproduction step (test, curl, script)
2. Check if the finding is a false positive → if so, move to the fix report as false positive, do not change code
3. Identify root cause, not just the symptom

### Phase 3 — Fix Implementation
1. Fix the root cause while respecting existing architecture boundaries
2. Make minimal, scoped changes — do not refactor unrelated code
3. Add or update tests for the fixed behavior:
   - Unit test for the specific logic fix
   - Integration test if DB/external service involved
   - Regression test to prevent re-occurrence
4. Run the build and relevant tests after each fix
5. Update `../wiki/{target-project-name}/00-single-source-of-truth.md` **only if** code changes make the wiki stale

### Phase 4 — Verification
1. Run the full test suite for affected modules
2. Run linter and type checker
3. Re-check the fixed flow with the original reproduction step
4. Verify no regressions in adjacent functionality

## Output

Write the fix report to:

**`prompts/report/check/full/report-fix-full-check.md`**

### Report Structure
```
# Fix Report — Full Check

## Summary
- Total findings: N
- Fixed: N
- False positives: N
- Deferred: N (with rationale)
- Files changed: N

## Fixed Items (sorted by severity)
| # | Finding | Severity | Root Cause | Files Changed | Verification |
|---|---------|----------|------------|---------------|--------------|

## False Positives
| Finding | Reason for dismissal | Evidence |
|---------|---------------------|----------|

## Deferred Items
| Finding | Rationale | Owner | Target fix date |
|---------|-----------|-------|-----------------|

## Verification Commands
- Command 1: <output>
- Command 2: <output>

## Residual Risk
- What was NOT fixed and why it is acceptable

## Follow-up Work
- Additional test coverage needed
- Monitoring/alerting to add
```

### Report Content Rules
- Every fixed item must reference the original finding from the check result
- List exact files changed with paths relative to project root
- Include verification command output (pass/fail) for each fix
- False positives must include detailed evidence explaining why they are not real defects
- Deferred items must include a clear business/technical rationale and an owner
