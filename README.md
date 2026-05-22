# System Prompt QA Suite

A collection of structured prompts for automated QA checking, security scanning, and fix execution on software projects.

## Directory Structure

```text
G:/DA/
|-- wiki/
|   `-- 00-single-source-of-truth.md        # Outside this prompt-suite repo
`-- system-prompt-qa-suite/
    |-- system_prompt/
    |   |-- 00-create-single-source-of-truth.md -> ../wiki/{target-project-name}/00-single-source-of-truth.md
    |   |-- 001-small-scan.md                  -> prompts/result/scan/small/result-small-scan.md
    |   |-- 002-small-check.md                 -> prompts/result/check/small/result-small-check.md
    |   |-- 003-full-scan.md                   -> prompts/result/scan/full/result-full-scan.md
    |   `-- 004-full-check.md                  -> prompts/result/check/full/result-full-check.md
    `-- prompts/
        |-- scan/
        |   |-- small/
        |   `-- full/
        |-- check/
        |   |-- small/
        |   `-- full/
        |-- fix/
        |   |-- scan/
        |   |   |-- small/
        |   |   `-- full/
        |   `-- check/
        |       |-- small/
        |       `-- full/
        |-- result/
        |   |-- scan/
        |   |   |-- small/
        |   |   `-- full/
        |   `-- check/
        |       |-- small/
        |       `-- full/
        `-- report/
            |-- scan/
            |   |-- small/
            |   `-- full/
            `-- check/
                |-- small/
                `-- full/
```

## Workflow

1. **Source of truth** - Always create or read `../wiki/{target-project-name}/00-single-source-of-truth.md` first.
2. **Small Scan** - `001-small-scan.md` runs `prompts/scan/small/`, then writes `prompts/result/scan/small/result-small-scan.md`.
3. **Small Check** - `002-small-check.md` runs `prompts/check/small/`, then writes `prompts/result/check/small/result-small-check.md`.
4. **Full Scan** - `003-full-scan.md` runs `prompts/scan/full/`, then writes `prompts/result/scan/full/result-full-scan.md`.
5. **Full Check** - `004-full-check.md` runs `prompts/check/full/`, then writes `prompts/result/check/full/result-full-check.md`.
6. **Fix Scan** - matching prompts under `prompts/fix/scan/{small|full}/` read from `prompts/result/scan/{small|full}/` and write reports to `prompts/report/scan/{small|full}/`.
7. **Fix Check** - matching prompts under `prompts/fix/check/{small|full}/` read from `prompts/result/check/{small|full}/` and write reports to `prompts/report/check/{small|full}/`.
8. **Loop** - repeat scan/check -> fix until all issues are resolved.

## Output Contract

| Phase | Prompt | Reads | Writes |
|---|---|---|---|
| Source of Truth | `system_prompt/00-create-single-source-of-truth.md` | target repository | `../wiki/{target-project-name}/00-single-source-of-truth.md` |
| Small Scan | `system_prompt/001-small-scan.md` | `../wiki/{target-project-name}/00-single-source-of-truth.md` | `prompts/result/scan/small/result-small-scan.md` |
| Small Check | `system_prompt/002-small-check.md` | `../wiki/{target-project-name}/00-single-source-of-truth.md` | `prompts/result/check/small/result-small-check.md` |
| Full Scan | `system_prompt/003-full-scan.md` | `../wiki/{target-project-name}/00-single-source-of-truth.md` | `prompts/result/scan/full/result-full-scan.md` |
| Full Check | `system_prompt/004-full-check.md` | `../wiki/{target-project-name}/00-single-source-of-truth.md` | `prompts/result/check/full/result-full-check.md` |
| Fix Small Scan | `prompts/fix/scan/small/00-fix-small-scan.md` | `prompts/result/scan/small/result-small-scan.md` | `prompts/report/scan/small/report-fix-small-scan.md` |
| Fix Small Check | `prompts/fix/check/small/00-fix-small-check.md` | `prompts/result/check/small/result-small-check.md` | `prompts/report/check/small/report-fix-small-check.md` |
| Fix Full Scan | `prompts/fix/scan/full/00-fix-full-scan.md` | `prompts/result/scan/full/result-full-scan.md` | `prompts/report/scan/full/report-fix-full-scan.md` |
| Fix Full Check | `prompts/fix/check/full/00-fix-full-check.md` | `prompts/result/check/full/result-full-check.md` | `prompts/report/check/full/report-fix-full-check.md` |

## Routing Rules

- `scan/` is for security, virus, malware, security audit, vulnerability scan, dependency risk, abuse cases, and other safety checks.
- `check/` is for non-security logic, configuration, workflow, consistency, system structure, quality, reliability, and product-readiness checks.
- `small/` contains fast baseline coverage, roughly 10-15 primary prompts.
- `full/` contains broader and deeper coverage across the whole system.
