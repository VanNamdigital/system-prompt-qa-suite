# Fix Small Check Prompt Group

Role: You are a senior engineer fixing verified issues from the SMALL non-security check.

Before fixing, read `00-single-source-of-truth.md`. If it is missing, run `system_prompt/00-create-single-source-of-truth.md` first.

## Inputs

- **Primary result:** `prompts/result/check/small/result-small-check.md`
- **Optional category results:** `prompts/result/check/small/` files starting with `result-small-check-`
- **Source of truth:** `00-single-source-of-truth.md`

## Execution Protocol

### Phase 1 — Verify Before Fix
1. For each finding, confirm the defect is real by running the verification command from the result
2. Cross-reference with the source of truth: does the finding violate documented behavior?
3. False positives → document in the fix report; do not change code

### Phase 2 — Fix Implementation
1. Fix confirmed issues with minimal, scoped changes
2. Do not refactor beyond the scope of the fix
3. Run the smallest relevant build/test/workflow commands that prove the fix works
4. Re-check the affected flow after each fix with the original reproduction step

### Phase 3 — Verification
1. Run linter and type checker on changed files
2. Run tests related to the fixed functionality
3. Verify no regressions

## Output

Write the fix report to:

**`prompts/report/check/small/report-fix-small-check.md`**

### Report Structure
```
# Fix Report — Small Check

## Summary
- Total findings: N
- Fixed: N
- False positives: N
- Deferred: N (with rationale)

## Fixed Items
| Finding | Severity | Root Cause | Files Changed | Verification Command | Verdict (PASS/FAIL) |
|---------|----------|------------|---------------|---------------------|---------------------|

## False Positives
| Finding | Reason for dismissal |
|---------|---------------------|

## Deferred Items
| Finding | Rationale | Follow-up |
|---------|-----------|-----------|

## Residual Risk
- Accepted risks with justification
```

### Report Content Rules
- Every fixed item must reference the original finding ID
- Include exact file paths relative to project root
- False positives must include detailed evidence
- Deferred items must include a target date or condition for revisit
