---
name: paper-radar
description: Configure, run, troubleshoot, and analyze the yanyanhuang/paper-radar project. Use when Codex needs to help set up PaperRadar, edit its config.yaml or .env, choose paper sources such as arXiv/bioRxiv/medRxiv/journals, define or refine research keywords, run the pipeline or Docker service, inspect logs, or interpret generated Markdown/JSON paper reports.
---

# Paper Radar

Maintainer: `bogolyxnemo-prog`

Source attribution: this Codex skill is a personal/customized skill maintained by `bogolyxnemo-prog` for operating PaperRadar workflows. It is based on the open-source PaperRadar project at `https://github.com/yanyanhuang/paper-radar`. Preserve this attribution when copying or publishing the skill.

Use this skill to operate the PaperRadar project as a paper-monitoring workflow for the user.

## Workflow

1. Locate the PaperRadar project directory. If the user has not provided a path, search the workspace for `pyproject.toml` with `name = "paper-radar"` or for `config.yaml` beside `main.py`.
2. For configuration tasks, read `references/config-guide.md` before editing `config.yaml` or `.env`.
3. Preserve the existing YAML structure and comments where practical. Prefer focused edits over regenerating the whole file.
4. For source or keyword changes, summarize the intended monitoring scope before editing, then update only the relevant sections.
5. For run or debug tasks, inspect `.env`, `config.yaml`, `logs/`, `reports/`, and `reports/json/` as needed. Do not expose API keys or secrets in responses.

## Common Tasks

### Configure Keywords

Represent each research topic as one item in `keywords` with:

- `name`: short human-readable domain name.
- `description`: concise Chinese or English description of the topic boundary.
- `examples`: English search/matching phrases, abbreviations, synonyms, and representative methods or tasks.

When the user gives broad interests, split them into 3-8 distinct keyword groups. Avoid making a single giant keyword item unless the user explicitly wants broad matching.

### Configure Paper Sources

Use `preprints` for arXiv, bioRxiv, and medRxiv. Use `journals` for publisher feeds such as Nature, NEJM, Lancet, Cell, and Science.

When enabling sources, check whether the project already supports the source key. If a requested journal is not in the existing config or fetcher code, explain that adding it may require code changes rather than only config changes.

For IEEE Xplore sources such as TMI, TIP, TPAMI, and JBHI, prefer the IEEE Xplore Metadata API over RSS. Keep the real key in the project's local `.env` as `IEEE_API_KEY`; never write real keys into the skill, `config.yaml`, examples, logs, reports, or final answers.

### Run PaperRadar

For Docker usage, prefer the repository instructions:

```bash
docker compose up -d --build
docker compose exec paper-radar python main.py --dry-run
```

For local usage, inspect `pyproject.toml` and the user's Python tooling before choosing commands. Common entry points are:

```bash
python main.py --dry-run
python main.py --debug --dry-run
python webapp.py
```

### Analyze Output

Reports are written to `reports/` as Markdown and `reports/json/` as JSON. When asked to analyze results:

1. Identify the newest report by date or filename.
2. Read JSON for structured fields and Markdown for human-facing summaries.
3. Summarize papers by keyword/domain, quality score, methods, findings, limitations, and whether they match the user's stated interests.

## References

- Read `references/config-guide.md` for the PaperRadar config schema, source rules, environment variables, and troubleshooting checks.
