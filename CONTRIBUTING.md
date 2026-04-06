# Contributing Guide

## First-time setup
1. Accept the GitHub collaborator invitation (check your email)
2. Clone the repo: `git clone https://github.com/Tauraslu/lab.git`
3. Never work directly on `main` — always create a branch

## Every time you start work
```bash
git pull origin main                         # get latest changes first
git checkout -b feature/your-task-name       # create your branch
```

## Branch naming
| Prefix | Use for |
|---|---|
| `feature/` | New additions |
| `fix/` | Corrections |
| `analysis/` | R or Python scripts |
| `docs/` | README or manual updates |

Examples:
- `feature/coding-batch-1-cheyenne`
- `analysis/irr-kappa-script`
- `fix/paper007-module-version`

## Commit message format
```
type: short description

Examples:
  coding: add papers 001-010 to tracking sheet
  analysis: add IRR kappa script
  fix: correct module_version for paper_007
  docs: update coding manual edge cases
```

## Submitting your work
```bash
git add .
git commit -m "coding: add papers 001-010"
git push origin feature/coding-batch-1-cheyenne
```
Then go to GitHub → open a Pull Request → assign Tauraslu as reviewer.

## File naming for papers
```
paper_###_Author_Year.pdf
```
Store in `papers/6item/` or `papers/10_18item/`

## Never commit
- Raw NHANES data files
- Paper PDFs (track in paper_tracking_sheet.csv only)
- `.DS_Store` or system files
- Any file with passwords or API keys
