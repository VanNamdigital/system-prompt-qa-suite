# System Prompt QA Suite

A collection of structured prompts for automated QA checking and fix execution on software projects.

## Directory Structure

```
system-prompt-qa-suite/
├── system_prompt/           # Orchestrator prompts (run in order)
│   ├── 00-create-single-source-of-truth.md   → wiki/00-source-of-truth.md
│   ├── 01-run-all-checks.md                  → prompts/result/result-*.md
│   └── 02-run-all-fixes.md                   → prompts/report/report-*.md
├── prompts/
│   ├── check/               # QA check prompts
│   ├── fix/                 # Fix prompts (paired with check)
│   ├── result/              # Check outputs
│   └── report/              # Fix reports
├── wiki/
│   └── 00-source-of-truth.md
└── .gitignore
```

## Workflow

1. **System prompt** → `00-create-single-source-of-truth.md` generates project documentation into `wiki/`
2. **Check** → `01-run-all-checks.md` runs all prompts in `prompts/check/`, outputs results to `prompts/result/`
3. **Fix** → `02-run-all-fixes.md` reads results, runs all prompts in `prompts/fix/`, writes reports to `prompts/report/`
4. **Loop** → Repeat check → fix until all issues are resolved
