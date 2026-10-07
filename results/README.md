# Results provenance

Raw [MindTrial](https://github.com/petmal/MindTrial) outputs for the CI/CD
Koan Benchmark. The article and the
root [`README.md`](../README.md) summarize these files; this page says
**which files produced which published figure** and which runs are kept
because their failure mode is itself a finding.

## Layout

```
results/
  koans-claude/YYYY-MM-DD/results-claude-HH-MM-SS.{csv,jsonl,html}
  koans-chat/…
  koans-gemini/…
  run.log                        # per-suite harness log (under each suite dir)
```

Each run directory contains:

| File | Use |
|---|---|
| `*.csv` | Per-task rows: status, duration, tokens, estimated cost, answer |
| `*.jsonl` | Same data plus end-of-file per-run summaries (`pass_rate_pct`, totals) |
| `*.html` | Human-readable MindTrial report |

Columns include `Status`, `Duration`, `InputTokens`, `OutputTokens`, and
`EstCostUSD`, so any published number can be recomputed.

## Canonical runs (scoreboards in root README)

Full 8-task arms used for the overall / stage / hardest-case tables.
Later dated files overwrite earlier ones when the same model arm appears
in more than one file (e.g. models added after the first sweep).

### Claude suite — `results/koans-claude/`

| File | What it adds |
|---|---|
| `2026-09-24/results-claude-10-30-37.*` | Core Anthropic / OpenAI / Google study + control arms |
| `2026-09-24/results-claude-10-48-21.*` | DeepSeek V3.2, Grok 4.6 |
| `2026-09-24/results-claude-14-39-23.*` | Claude Opus 5.5 |
| `2026-09-29/results-claude-15-13-10.*` | Claude Fable 5.1, GPT-6 Astra |

### ChatGPT suite — `results/koans-chat/`

| File | What it adds |
|---|---|
| `2026-09-24/results-chat-10-55-21.*` | Core study + control arms |
| `2026-09-24/results-chat-14-44-18.*` | Claude Opus 5.5 |
| `2026-09-29/results-chat-15-16-06.*` | Claude Fable 5.1, GPT-6 Astra |

### Gemini suite — `results/koans-gemini/`

| File | What it adds |
|---|---|
| `2026-09-24/results-gemini-11-08-21.*` | Core study + control arms |
| `2026-09-24/results-gemini-14-47-05.*` | Claude Opus 5.5 |
| `2026-09-29/results-gemini-15-18-31.*` | Claude Fable 5.1, GPT-6 Astra |

Earlier directories under `2026-09-16/` and `2026-09-17/` are partial or
pre-freeze sweeps (some only 5 tasks). They are retained for history; they
are **not** in the scoreboards above.

## Koan 09

Koan 09 (tag filters) is the hardest case in the study. Failures consistently
select `fix_target_job: deploy` instead of `all_jobs`. See the root
[`README.md`](../README.md#prompt-wording-can-change-the-outcome-koan-09) and
[`docs/koan-answers-README.md`](../docs/koan-answers-README.md).

## Expected-result contract

The locked field values every task must match are listed in the root
[`README.md`](../README.md#locked-answer-keys) and fully explained in
[`docs/koan-answers-README.md`](../docs/koan-answers-README.md).

| Koan case | Stage | Field | Expected |
|---|---|---|---|
| **03 Databases** | Test | `root_cause` | `no_postgres_service_container` |
| | | `fix` | `add_postgres_service_image` |
| **04 Parallelism** | Test | `suite_split` | `false` |
| | | `each_node_ran_full_suite` | `true` |
| | | `fix` | `use_circleci_tests_split` |
| **05 Orbs** | Build | `install_attempted` | `false` |
| | | `fix` | `add_ruby_orb` |
| **06 Caching** | Build | `cache_restored` | `false` |
| | | `checksum_target` | `package.json` |
| | | `fix` | `enable_restore_and_save_cache` |
| **07 Test Summary** | Test | `results_produced` | `true` |
| | | `job_green` | `false` |
| | | `missing_store_test_results` | `true` |
| | | `missing_store_artifacts` | `true` |
| | | `fix` | `add_store_test_results_and_store_artifacts` |
| **08 Branch Filters** | Deploy | `pipeline_green` | `true` |
| | | `behaviour_correct` | `false` |
| | | `fix_target_job` | `deploy` |
| | | `fix` | `filters_branches_only_production` |
| **09 Tag Filters** | Deploy | `pipeline_green` | `true` |
| | | `behaviour_correct` | `false` |
| | | `fix_target_job` | `all_jobs` |
| | | `fix` | `filters_tags_only_beta` |
| **13 Workspaces** | Build | `builds_green` | `true` |
| | | `deploy_green` | `false` |
| | | `workspace_had_apps` | `false` |
| | | `fix` | `persist_app_paths` |

### Note on Koans 08 and 09

These two cases look structurally identical but have different answers for
`fix_target_job`. In 08 (branch filters) jobs already run on every branch, so
filtering `deploy` alone works. In 09 (tag filters) a tag push runs a job only
if that job carries its own tag filter — filtering `deploy` alone breaks the
`requires` chain and nothing runs, so the answer is `all_jobs`.
