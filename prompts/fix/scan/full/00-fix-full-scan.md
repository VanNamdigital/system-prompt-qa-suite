# Fix Full Scan Prompt Group

Role: You are a senior engineer fixing verified issues from the FULL security scan.

Before fixing, read `wiki/00-single-source-of-truth.md`. If it is missing, run `system_prompt/00-create-single-source-of-truth.md` first.

## Inputs

- **Primary result:** `prompts/result/scan/full/result-full-scan.md`
- **Optional category results:** `prompts/result/scan/full/` files starting with `result-full-scan-`
- **Source of truth:** `wiki/00-single-source-of-truth.md`

## Execution Protocol

### Phase 1 — Triage by Severity
Prioritize fixes in this order:
1. **CRITICAL**: Remote code execution, credential exposure (live), SQL injection, auth bypass — fix immediately
2. **HIGH**: XSS, IDOR, SSRF, insecure deserialization, hardcoded secrets — fix within sprint
3. **MEDIUM**: Missing rate limit, permissive CORS, verbose errors — fix within 2 sprints
4. **LOW**: Missing security headers, cookie flags — fix within a month
5. **INFO**: Observations, hardening suggestions — backlog

### Phase 2 — Reproduce Before Fix
For each finding:
1. Confirm the vulnerable path exists (read the code, trace the input/output)
2. Reproduce with a deterministic command (curl, script, unit test)
3. If unable to reproduce → document as false positive; do not change code
4. For CRITICAL/HIGH: verify the finding is not blocked by other controls (WAF, firewall, input validation)

### Phase 3 — Fix Root Cause
1. Fix the root cause, not only the shown example — same vulnerability pattern may exist elsewhere
2. For each fix:
   - Apply the security fix (sanitization, authorization, encryption, etc.)
   - Add regression tests for the vulnerable path (positive + negative cases)
   - Ensure no breaking changes to public API contracts
3. For CRITICAL/HIGH findings: check for similar patterns across the codebase (batch fix)
4. Run code linter and type checker

### Phase 4 — Re-scan and Verify
1. Run relevant build/test/security verification commands
2. Re-scan the fixed areas for regression and bypass variants
3. For auth/access control fixes: verify legitimate users still have access
4. For crypto/token fixes: verify existing tokens/sessions still work (or have graceful migration)

## Output

Write the fix report to:

**`prompts/report/scan/full/report-fix-full-scan.md`**

### Report Structure
```
# Security Fix Report — Full Scan

## Summary
- Total findings: N
- Fixed: N
- False positives: N
- Deferred: N (with rationale)
- Accepted risk: N (with justification)
- Files changed: N

## Fixed Items (by severity)
| # | Severity | Finding ID | Root Cause | Files Changed | Verification Command | Verdict |
|---|----------|------------|------------|---------------|---------------------|---------|

## False Positives
| Finding ID | Reason for dismissal | Original Evidence Reviewed |
|------------|---------------------|---------------------------|

## Accepted Risk (Not Fixed)
| Finding ID | Rationale | Compensating Controls | Review Date |
|------------|-----------|----------------------|-------------|

## Deferred Items
| Finding ID | Rationale | Owner | Target Sprint |
|------------|-----------|-------|---------------|

## Verification Commands Executed
- Command 1: <output>
- Command 2: <output>

## Regression Check
- Areas re-scanned: N files
- New findings introduced: N (if any)
- Bypass variants tested: N (list)
```

### Report Content Rules
- Every fixed item must reference the finding ID from the scan result
- Include exact file paths relative to project root
- For accepted risk: describe compensating controls (e.g., "protected by WAF", "requires admin authentication")
- For false positives: include the exact code/evidence reviewed
