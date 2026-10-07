# CI/CD Koan Benchmark

Companion repository for **Benchmarking Models Across CI/CD Stages: First
Results, and What We Learned**.

The article is the readable version. This repo is the evidence trail: the
frozen CircleCI failure cases, the human-locked answer keys, the three
independently authored prompt suites, and the raw
[MindTrial](https://github.com/petmal/MindTrial) scores behind every published
figure.

**15 models × 8 frozen cases × 3 prompt suites = 24 observations per model.**
Every case comes from a pipeline that actually failed. Every answer key comes
from human feedback and CircleCI Training Koans — not from a model — so
authoring models never decide what “correct” means.

---

## How to use this repo

| If you want to… | Go to |
|---|---|
| See the scoreboards | [Results](#results) below |
| Inspect the evidence each task is built from | [`corpus/`](corpus/) |
| Check an expected answer and why it is correct | [`docs/koan-answers-README.md`](docs/koan-answers-README.md) |
| See how the three suites were authored | [`docs/koan-task-generation-prompt.md`](docs/koan-task-generation-prompt.md) |
| Read the actual task prompts | [`tasks-koans-claude.yaml`](tasks-koans-claude.yaml), [`tasks-koans-chat.yaml`](tasks-koans-chat.yaml), [`tasks-koans-gemini.yaml`](tasks-koans-gemini.yaml) |
| Dig into per-task scores, cost, and latency | [`results/`](results/) · provenance in [`results/README.md`](results/README.md) |
| Reproduce a run | [`config-koans.yaml`](config-koans.yaml) + [MindTrial](https://github.com/petmal/MindTrial) |

---

## Scope

| | |
|---|---|
| Observations per model | **24** (8 frozen cases × 3 author suites) |
| Runs per observation | Once (no repeated trials) |
| Pass-rate grain | One task ≈ **4.17** points of pass rate |
| Build | 3 cases (05, 06, 13) × 3 suites = **9** |
| Test | 3 cases (03, 04, 07) × 3 suites = **9** |
| Deploy | 2 cases (08, 09) × 3 suites = **6** |

Models diagnosed **frozen** pipeline failures. They read a config, a job log,
and a workflow summary, then answered structured questions. They did not edit
configs, re-run pipelines, or fix anything live. A sibling benchmark,
[pipeline-fix-rerun-bench](https://github.com/CircleCI-Research/pipeline-fix-rerun-bench),
grades agents on actual repair with a live rerun as the score — a different
capability, and the rankings diverge.

### Grading

Every task returns a JSON object against a fixed schema (booleans and closed
enums, never free text). Scoring is an **exact match** on every required field.
No model judge is involved. Enum options include plausible-but-wrong answers so
symptom fixes do not get partial credit.

Answer keys are locked from CircleCI’s published
[Training Koans](https://github.com/CircleCI-Support/CircleCI-Training-Koans),
frozen as real failed pipelines, and recorded **before** any task was
generated. Authoring models were given the locked answer and instructed to
copy it; any generated task that differed was discarded.

Full keys and distractors: [`docs/koan-answers-README.md`](docs/koan-answers-README.md).

### Cost

Each response includes input and output token counts. Cost per task is those
counts × the model’s published per-million rates.

- **$/task** = total spend across 24 observations ÷ 24  
- **$/correct** = total spend ÷ tasks passed (wrong answers still cost money)

For example, Claude Opus 5.5 on koan-03 used 2,396 input and 468 output tokens
→ about $0.0189 for that one task. Opus 5.5’s `/task` and `/correct` are
identical because it never failed; DeepSeek’s rise from $0.0013 to $0.0022
reflects failures it still paid for.

---

## Results

Recompute any figure from the CSV/JSONL under [`results/`](results/) (see
[`results/README.md`](results/README.md) for which files feed which table).

### Breakdown across all koan suites

| Model | Provider | Build | Test | Deploy | Pass rate | Median | $/task | $/correct |
|---|---|---|---|---|---|---|---|---|
| Claude Opus 5.5 | Anthropic | 9/9 | 9/9 | 6/6 | **100%** | 6.9s | $0.0182 | $0.0182 |
| Claude Fable 5.1 | Anthropic | 9/9 | 9/9 | 5/6 | 95.83% | 8.6s | $0.0433 | $0.0451 |
| Claude Opus 5 | Anthropic | 9/9 | 9/9 | 5/6 | 95.83% | 8.6s | $0.0234 | $0.0244 |
| GPT-5.6 Sol | OpenAI | 9/9 | 9/9 | 5/6 | 95.83% | 4.6s | $0.0157 | $0.0164 |
| GPT-6 Astra | OpenAI | 9/9 | 9/9 | 5/6 | 95.83% | 11.3s | $0.0308 | $0.0322 |
| GPT-5.6 Luna | OpenAI | 8/9 | 9/9 | 6/6 | 95.83% | 4.3s | $0.00088 | $0.00092 |
| Gemini 3.1 Pro | Google | 8/9 | 9/9 | 6/6 | 95.83% | 9.3s | $0.0038 | $0.0040 |
| GPT-5.6 Terra | OpenAI | 7/9 | 8/9 | 6/6 | 87.50% | 4.3s | $0.0067 | $0.0076 |
| Grok 4.6 | xAI (via Bedrock) | 9/9 | 7/9 | 5/6 | 87.50% | 12.0s | $0.0146 | $0.0167 |
| Claude Sonnet 5 | Anthropic | 8/9 | 8/9 | 4/6 | 83.33% | 6.4s | $0.0094 | $0.0112 |
| GPT-5 Mini (high) | OpenAI | 7/9 | 8/9 | 4/6 | 79.17% | 32.5s | $0.0223 | $0.0282 |
| Claude Haiku 4.5 | Anthropic | 6/9 | 8/9 | 3/6 | 70.83% | 5.2s | $0.0033 | $0.0047 |
| GPT-5 Mini (low) | OpenAI | 7/9 | 8/9 | 2/6 | 70.83% | 5.6s | $0.0042 | $0.0059 |
| Gemini 2.5 Flash | Google | 5/9 | 8/9 | 3/6 | 66.67% | 8.1s | $0.0003 | $0.0005 |
| DeepSeek V3.2 | DeepSeek (via Bedrock) | 6/9 | 6/9 | 2/6 | 58.33% | 8.7s | $0.0013 | $0.0022 |

**Pass rate** = pass / 24 attempted. **Median** = middle duration across those
24 tasks. When pass rates tie, cost and speed decide which model fits your
pipeline priorities.

### By stage (tied leaders)

When several models tie on stage pass count, they are listed with per-stage
`$/correct` and median speed:

| Stage | Model | Provider | Per-stage $/correct | Median |
|---|---|---|---|---|
| **Build** | GPT-5.6 Sol | OpenAI | $0.0115 | 4.6s |
| | Claude Opus 5.5 | Anthropic | $0.0161 | 6.4s |
| | Grok 4.6 | xAI (via Bedrock) | $0.0111 | 6.8s |
| | Claude Opus 5 | Anthropic | $0.0196 | 8.6s |
| | Claude Fable 5.1 | Anthropic | $0.0413 | 9.9s |
| | GPT-6 Astra | OpenAI | $0.0298 | 11.2s |
| **Test** | GPT-5.6 Sol | OpenAI | $0.0148 | 4.2s |
| | GPT-5.6 Luna | OpenAI | $0.00074 | 3.8s |
| | Claude Opus 5 | Anthropic | $0.0235 | 8.2s |
| | Claude Opus 5.5 | Anthropic | $0.0188 | 7.7s |
| | Gemini 3.1 Pro | Google | $0.0043 | 8.9s |
| | Claude Fable 5.1 | Anthropic | $0.0403 | 7.9s |
| | GPT-6 Astra | OpenAI | $0.0310 | 9.7s |
| **Deploy** | Gemini 3.1 Pro | Google | $0.0035 | 16.6s |
| | GPT-5.6 Terra | OpenAI | $0.0109 | 8.4s |
| | GPT-5.6 Luna | OpenAI | $0.0017 | 9.7s |
| | Claude Opus 5.5 | Anthropic | $0.0205 | 8.3s |
| **Best value** (all 24) | GPT-5.6 Luna | OpenAI | $0.00092 | 4.3s |
| **Best accuracy** (all 24) | Claude Opus 5.5 | Anthropic | $0.0182 | 6.9s |

### Prompt wording can change the outcome (koan-09)

On **koan-09** (tag filters), pass rates ranged from **12/15** to **4/15**
across the three suites. Frozen artifacts and locked answers were identical;
only the question wording differed.

**Locked answer:** `all_jobs`. On CircleCI, a tag push runs a job only if that
job has its own tag filter. Put the filter on `deploy` alone and build/test
never start on the tag, so the `requires` chain never fires. Every miss
answered `deploy`.

| Suite author | What the task states | Passed | Failed |
|---|---|---|---|
| ChatGPT | “the build, test, and deploy jobs must participate in that tagged flow… Determine the affected workflow scope…” | **12 of 15** | DeepSeek V3.2, GPT-5 Mini (low), Gemini 2.5 Flash |
| Claude | “the deploy job must run only for tagged commits… which job(s) need their workflow configuration changed?” | **9 of 15** | DeepSeek V3.2, GPT-5 Mini (low/high), Gemini 2.5 Flash, Claude Haiku 4.5, Claude Sonnet 5 |
| Gemini | “we only ever want to run our deploy job when we commit a tag… Ensure the deploy job only runs for tagged commits…” | **4 of 15** | DeepSeek V3.2, GPT-5 Mini (low/high), Gemini 2.5 Flash, Claude Haiku 4.5, Claude Sonnet 5, Claude Fable 5.1, Claude Opus 5, GPT-5.6 Sol, GPT-6 Astra, Grok 4.6 |

Only four models cleared koan-09 on **all three** suites: Claude Opus 5.5,
GPT-5.6 Terra, GPT-5.6 Luna, and Gemini 3.1 Pro.

**Author check.** Sharing a family with the suite author did not hand anyone a
free pass: Haiku and Sonnet still failed Claude-written tasks; Gemini 2.5 Flash
still failed Gemini-written tasks; Gemini 3.1 Pro’s only miss of 24 was on the
suite it wrote (koan-06, Caching).

Task files: [`tasks-koans-chat.yaml`](tasks-koans-chat.yaml) ·
[`tasks-koans-claude.yaml`](tasks-koans-claude.yaml) ·
[`tasks-koans-gemini.yaml`](tasks-koans-gemini.yaml).

### TL;DR

| Priority | Pick | Why |
|---|---|---|
| **Best value** | GPT-5.6 Luna | 23/24 at $0.00092 per correct; fastest median (4.3s); one of four models to clear koan-09 on every suite |
| **Best accuracy** | Claude Opus 5.5 | Only **24/24**; `$/task` = `$/correct`; one-task lead over five models tied at 23/24 |

The accuracy gap is one task for roughly **20×** the cost per correct answer.
How you weigh that is the point of publishing both columns.

---

## Locked answer keys

Every generated task’s `expected-result` must match the contract below.

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

Rationale and incorrect-fix distractors:
[`docs/koan-answers-README.md`](docs/koan-answers-README.md).

**08 vs 09.** They look similar but disagree on `fix_target_job`. In 08, jobs
already run on every branch, so filtering `deploy` alone works. In 09,
filtering `deploy` alone breaks `requires` on a tag push — answer is
`all_jobs`.

---

## The eight frozen cases

```
corpus/koans-09-tag-filters/
  config.yml        .circleci/config.yml as it stood when the pipeline ran
  workflow.txt      observed workflow state: branch, jobs, statuses
  koan-README.md    public CircleCI Training Koan this derives from
  source.md         pipeline number, frozen branch, stated requirement
```

| Case | Stage | What’s wrong |
|---|---|---|
| [03 Databases](corpus/koans-03-databases/) | Test | No Postgres service container; readiness check times out |
| [04 Parallelism](corpus/koans-04-parallelism/) | Test | `parallelism: 2` is set but the suite never splits |
| [05 Orbs](corpus/koans-05-orbs/) | Build | Empty `orbs:` block; `ruby -v` exits 127 |
| [06 Caching](corpus/koans-06-caching/) | Build | Cache steps commented out; `yarn install --offline` fails |
| [07 Test Summary](corpus/koans-07-test-summary/) | Test | JUnit written but never published |
| [08 Branch Filters](corpus/koans-08-branch-filters/) | Deploy | Green pipeline; deploy ran on a non-production branch |
| [09 Tag Filters](corpus/koans-09-tag-filters/) | Deploy | Green pipeline; deploy ran on a branch push, not a beta tag |
| [13 Workspaces](corpus/koans-13-workspaces/) | Build | Builds persist the wrong paths; deploy finds no apps |

When the freeze and the public Koan README disagree, **ground truth follows the
freeze**.

---

## Models

| Model | Provider | Route | Config | Role |
|---|---|---|---|---|
| Claude Fable 5.1 | Anthropic | direct | `effort: high` | study |
| Claude Opus 5.5 | Anthropic | direct | `effort: high` | study |
| Claude Opus 5 | Anthropic | direct | `effort: high` | study |
| Claude Sonnet 5 | Anthropic | direct | `effort: high` | study / suite author |
| Claude Haiku 4.5 | Anthropic | direct | provider default | tier control |
| GPT-6 Astra | OpenAI | direct | `reasoning-effort: high` | study |
| GPT-5.6 Sol | OpenAI | direct | `reasoning-effort: high` | study |
| GPT-5.6 Luna | OpenAI | direct | `reasoning-effort: high` | study / suite author |
| GPT-5.6 Terra | OpenAI | direct | `reasoning-effort: high` | study |
| GPT-5 Mini | OpenAI | direct | `reasoning-effort: high` | effort arm |
| GPT-5 Mini | OpenAI | direct | `reasoning-effort: low` | effort arm |
| Gemini 3.1 Pro | Google | Vertex AI / ADC | `thinking-level: high` | study / suite author |
| Gemini 2.5 Flash | Google | Vertex AI / ADC | provider default | tier control |
| Grok 4.6 | xAI | Bedrock, OpenAI-compatible | `json-schema` | study |
| DeepSeek V3.2 | DeepSeek | Bedrock, OpenAI-compatible | `json-object` | family control |

Suite authors (high reasoning): Claude Sonnet 5, GPT-5.6 Luna, Gemini 3.1 Pro.
Authoring prompt: [`docs/koan-task-generation-prompt.md`](docs/koan-task-generation-prompt.md).

Provider wiring: [`config-koans.yaml`](config-koans.yaml) (template; no credentials).

---

## Repository layout

| Path | Contents |
|---|---|
| [`corpus/`](corpus/) | Eight frozen failure cases |
| [`docs/koan-answers-README.md`](docs/koan-answers-README.md) | Locked answer keys and distractors |
| [`docs/koan-task-generation-prompt.md`](docs/koan-task-generation-prompt.md) | Suite-authoring prompt |
| [`tasks-koans-*.yaml`](tasks-koans-claude.yaml) | Three MindTrial task suites |
| [`results/`](results/) | Per-task CSV / JSONL / HTML |
| [`config-koans.yaml`](config-koans.yaml) | Provider configuration template |

```bash
# Pick a suite in config-koans.yaml, then:
mindtrial --config=config-koans.yaml run
```
