# PaperRadar Configuration Guide

Use this guide when configuring the `bogolyxnemo-prog` customized PaperRadar workflow.

Attribution: this skill is maintained by `bogolyxnemo-prog` and is based on the upstream open-source project `https://github.com/yanyanhuang/paper-radar`.

## Project Shape

PaperRadar is a Python application for daily paper monitoring. It fetches papers from preprint and journal feeds, filters them with a light LLM, optionally parses PDFs with MinerU, analyzes papers with a heavy LLM, and writes Markdown plus JSON reports.

Important files:

- `config.yaml`: research topics, sources, model settings, MinerU, output, runtime, and EZproxy settings.
- `.env`: API bases, API keys, model names, timezone, optional library credentials.
- `main.py`: pipeline entry point.
- `webapp.py`: FastAPI web UI entry point.
- `reports/`: Markdown reports.
- `reports/json/`: JSON reports used by the web UI.
- `logs/`: runtime logs.

## Keywords

`keywords` is a YAML list. Each item should have:

```yaml
keywords:
  - name: "Medical Image Analysis"
    description: "Medical imaging AI, radiology, CT, MRI, ultrasound, X-ray."
    examples:
      - "medical image analysis, radiology AI, diagnostic imaging"
      - "CT, MRI, X-ray, ultrasound, mammography"
      - "medical image segmentation, detection, classification"
```

Guidelines:

- Keep `name` stable and readable because it appears in reports.
- Put domain boundaries in `description`.
- Put English query-like phrases, synonyms, abbreviations, methods, datasets, and task names in `examples`.
- Prefer several focused keyword groups over one broad group.
- Use Chinese descriptions when the user's working language is Chinese, but include English examples for paper matching.

## Preprint Sources

Preprint feeds belong under `preprints`.

```yaml
preprints:
  enabled: true
  arxiv:
    enabled: true
    categories: "cs.AI+cs.CV+cs.CL+cs.LG+eess.IV+q-bio.QM"
    max_papers_per_day: 0
    retry_interval_minutes: 30
    max_retry_hours: 5
  max_papers_per_source: 30
  sources:
    - name: "bioRxiv"
      key: "biorxiv"
      enabled: true
    - name: "medRxiv"
      key: "medrxiv"
      enabled: true
```

Rules:

- Use `preprints.enabled` as the global preprint switch.
- Use `preprints.arxiv.enabled` for arXiv.
- arXiv categories are joined with `+`, for example `cs.AI+cs.CV+cs.CL`.
- `max_papers_per_day: 0` means no local daily cap for arXiv.
- `bioRxiv` and `medRxiv` are preprint sources and should stay in `preprints.sources`, not `journals.sources`.

Common arXiv categories:

- AI/ML/NLP/CV: `cs.AI`, `cs.LG`, `cs.CL`, `cs.CV`, `stat.ML`
- Medical imaging/signal: `eess.IV`, `eess.SP`, `q-bio.QM`
- HCI/agents/systems: `cs.HC`, `cs.MA`, `cs.SE`, `cs.DC`
- Robotics: `cs.RO`

## Journal Sources

Journal feeds belong under `journals`.

```yaml
journals:
  enabled: true
  max_papers_per_journal: 30
  sources:
    - name: "Nature"
      key: "nature"
      enabled: true
    - name: "Nature Medicine"
      key: "nature_medicine"
      enabled: true
    - name: "Lancet Digital Health"
      key: "lancet_digital_health"
      enabled: true
```

Known source keys present in the reference project include:

- `nature`
- `nature_communications`
- `nature_medicine`
- `nature_methods`
- `nature_machine_intelligence`
- `nature_biomedical_engineering`
- `nature_computational_science`
- `nejm`
- `lancet`
- `lancet_digital_health`
- `cell`
- `cancer_cell`
- `cell_reports_medicine`
- `science`
- `science_advances`

If a user requests a new journal key that is not already supported by `journal_fetcher.py`, add or modify fetcher code instead of only adding a config entry.

## Environment Variables

`.env` should contain OpenAI-compatible model endpoints:

```dotenv
LIGHT_LLM_API_BASE=https://api.example.com/v1
LIGHT_LLM_API_KEY=...
LIGHT_LLM_MODEL=...

HEAVY_LLM_API_BASE=https://api.example.com/v1
HEAVY_LLM_API_KEY=...
HEAVY_LLM_MODEL=...

MINERU_API_BASE=https://mineru.net
MINERU_API_KEY=...
TZ=Asia/Shanghai
```

Optional EZproxy credentials:

```dotenv
HKU_LIBRARY_UID=...
HKU_LIBRARY_PIN=...
```

Never print real API keys or credentials in final answers. Refer to them by variable name.

## LLM and MinerU Settings

`llm.light` performs fast title/abstract filtering. `llm.heavy` performs full-paper analysis after MinerU conversion. `llm.summary` generates domain summaries and can use `light` or `heavy`.

Tune these fields cautiously:

- `temperature`: lower for deterministic filtering, slightly higher for summaries.
- `max_tokens`: keep heavy high enough for structured full-paper output.
- `rate_limit.requests_per_minute`: `0` means no local limit.

`mineru.enabled: true` enables PDF-to-Markdown parsing. Important fields:

- `mode: "auto"`: use precise mode when token is available, otherwise agent fallback.
- `language`: commonly `"ch"` for Chinese report needs.
- `cache_dir`: keep stable to avoid repeated parsing.
- `concurrent_parsing`: keep low on small VPS/NAS machines, usually 2-3.
- `max_markdown_chars`: cap Markdown passed to the heavy LLM.

## Output and Runtime

Outputs:

```yaml
output:
  language: "Chinese"
  formats:
    markdown:
      enabled: true
      path: "./reports/"
    json:
      enabled: true
      path: "./reports/json/"
```

Runtime:

```yaml
runtime:
  schedule: "0 14 * * *"
  timezone: "Asia/Shanghai"
  retry_count: 3
  pdf_timeout: 120
  concurrent_filtering: 5
  concurrent_analysis: 3
```

Use cron syntax for `schedule`. Confirm the container or host timezone when users ask about run time.

## Validation and Troubleshooting

Before running:

- Confirm `.env` exists and required variables are set.
- Confirm `config.yaml` has `keywords`, `preprints`, `journals`, `llm`, `mineru`, `output`, and `runtime`.
- Confirm `biorxiv` and `medrxiv` are under `preprints.sources`.
- Confirm requested journals use supported `key` values.

Useful commands:

```bash
python main.py --debug --dry-run
python main.py --test --dry-run
docker compose up -d --build
docker compose logs -f paper-radar
docker compose exec paper-radar python main.py --dry-run
```

Common failures:

- Missing API key: check `.env` and variable interpolation in `config.yaml`.
- Empty arXiv feed: check categories and retry settings.
- Journal PDFs inaccessible: disable `ezproxy.enabled` or provide valid credentials.
- MinerU timeout: reduce concurrency, increase `max_wait_seconds`, or temporarily disable MinerU.
- No reports generated: inspect `logs/`, ensure at least one paper matched a keyword, and check output paths.
