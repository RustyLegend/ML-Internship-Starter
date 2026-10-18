# Agent instructions

Before any task in this repo: **read `skills/README.md`** — it is the router.
Find the task in its table and load exactly **one** skill (plus `skills/flyrank/flyrank-data/SKILL.md`
whenever the task touches the data). Do not load every skill; keep context small.

Ground rules for this repo:
- Search the repo before assuming something is missing or not implemented.
- One task per conversation; finish and verify before starting the next.
- Never commit datasets (CI blocks them). Never print private data, client names, or raw queries.
- The intern validates your output — end each task by running the notebook top to bottom.

# Internship execution rules

- Follow the repository's assignment specification and acceptance criteria.
- Inspect the existing implementation before editing files.
- Work autonomously through implementation, testing, debugging, and final verification.
- Keep PROJECT_PROGRESS.md updated with completed work, changed files, commands, results, blockers, and next steps.
- Use targeted queries and samples for large datasets; do not dump entire datasets into the context window.
- Validate schemas, data quality, temporal splits, target leakage, baseline metrics, and ranking quality where applicable.
- Never fabricate results, test outcomes, citations, paper URLs, or deliverables.
- Run focused tests after each stage and the full pipeline after integration.
- Preserve unrelated user work and avoid destructive operations.
- Do not push commits, publish a paper, expose secrets, or make external submissions without explicit authorization.
- Do not claim completion until the acceptance criteria have been checked against actual outputs.
