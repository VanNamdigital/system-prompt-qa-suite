# Fix Small Scan Prompt Group

Role: You are a senior engineer fixing verified issues from the SMALL security scan.

Before fixing, read `../wiki/{target-project-name}/00-single-source-of-truth.md`. If it is missing, run `system_prompt/00-create-single-source-of-truth.md` first.

## Inputs

- **Primary result:** `prompts/result/scan/small/result-small-scan.md`
- **Optional category results:** `prompts/result/scan/small/` files starting with `result-small-scan-`
- **Source of truth:** `../wiki/{target-project-name}/00-single-source-of-truth.md`

## Execution Protocol

### Phase 1 — Verify Each Finding
1. Read each finding and confirm the vulnerable code path exists
2. Run a deterministic command to reproduce or verify the issue:
   - Grep the vulnerable pattern
   - Trace the input-to-sink path
   - If a test exists, run it
3. False positives → document in the fix report; do not change code

### Phase 2 — Fix Confirmed Issues
1. Fix with production-grade changes:
   - Security: parameterize, sanitize, authorize, encrypt, rate-limit
   - Quality: follow existing code patterns, add tests
2. Make minimal, scoped changes — do not introduce unrelated refactoring
3. Run the smallest relevant build/test/scan commands that prove the fix

### Phase 3 — Re-check
1. Re-read the fixed code and confirm the vulnerable path is closed
2. Run linter and type checker on changed files
3. Verify no regressions in adjacent functionality

## Output

Write the fix report to:

**`prompts/report/scan/small/report-fix-small-scan.md`**

### Report Structure
```
# Security Fix Report — Small Scan

## Summary
- Total findings: N
- Fixed: N
- False positives: N
- Deferred: N
- Residual risk: N

## Fixed Items
| Finding | Severity | Root Cause | Files Changed | Verification Command | Verdict |
|---------|----------|------------|---------------|---------------------|---------|

## False Positives
| Finding | Reason for dismissal | Evidence |
|---------|---------------------|----------|

## Deferred Items
| Finding | Rationale | Follow-up Action |
|---------|-----------|------------------|

## Residual Risk
- Vulnerabilities that were assessed but deemed acceptable or too costly to fix now
- Justification for each
```

### Report Content Rules
- Every fixed item must reference the original finding
- Include exact file paths relative to project root
- False positives must include detailed evidence explaining why they are not real issues
- Deferred items must include a follow-up action (e.g., "Create Jira ticket SEC-123", "Escalate to architect")
