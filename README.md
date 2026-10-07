# CI/CD Koan Benchmark
 
Data Benchmarking Models Across CI/CD Stages: First Results, and What We Learned
 
14 models answered 8 frozen CircleCI failure cases across 3
independently-authored prompt suites.

Every case goes back to a pipeline that actually failed, and every answer key is used human feedback and CircleCI's own training material rather than written by a
model to prevent model bias. 
 
The article is the readable version. This repo is where you check the fields, the expected answers, and the raw scores.
 
---
 
## Table of Contents
 
| If you want to... | Go to |
|---|---|
| See the evidence each task is built from | [`corpus/`](corpus/) |
| Check an expected answer and why it's correct | [`docs/koan-answers-README.md`](docs/koan-answers-README.md) |
| See how the three suites were authored | [`docs/koan-task-generation-prompt.md`](docs/koan-task-generation-prompt.md) |
| Read the actual task prompts | [`tasks-koans-claude.yaml`](tasks-koans-claude.yaml), [`-chat`](tasks-koans-chat.yaml), [`-gemini`](tasks-koans-gemini.yaml) |
| Dig into per-task scores, cost and latency | [`results/`](results/) |
 
---

## The 14 Models
| Model | Provider | Route | Config | Role |
|---|---|---|---|---|
| Claude Fable 5.1 | Anthropic | direct | `effort: high` | study |
| Claude Opus 5.5 | Anthropic | direct | `effort: high` | study |
| Claude Opus 5 | Anthropic | direct | `effort: high` | study |
| Claude Sonnet 5 | Anthropic | direct | `effort: high` | study |
| Claude Haiku 4.5 | Anthropic | direct | provider default | tier control |
| GPT-6 Astra | OpenAI | direct | `reasoning-effort: high` | study |
| GPT-5.6 Sol | OpenAI | direct | `reasoning-effort: high` | study |
| GPT-5.6 Terra | OpenAI | direct | `reasoning-effort: high` | study |
| GPT-5 Mini | OpenAI | direct | `reasoning-effort: high` | effort arm |
| GPT-5 Mini | OpenAI | direct | `reasoning-effort: low` | effort arm |
| Gemini 3.1 Pro | Google | Vertex AI / ADC | `thinking-level: high` | study (substitute) |
| Gemini 2.5 Flash | Google | Vertex AI / ADC | provider default | tier control |
| Grok 4.6 | xAI | Bedrock, OpenAI-compatible | `json-schema` | study (substitute) |
| DeepSeek V3.2 | DeepSeek | Bedrock, OpenAI-compatible | `json-object` | family control |

---

## What's in each folder
  
### `corpus/` — the eight frozen cases
 
One folder per case. Each case is a snapshot of a specific pipeline run, not
"whatever is red today," so every model sees byte-identical evidence.
 
```
corpus/koans-09-tag-filters/
  config.yml        the .circleci/config.yml as it stood when the pipeline ran
  workflow.txt      the observed workflow state: branch, jobs, statuses
  koan-README.md    the public CircleCI Training Koan this derives from
  source.md         pipeline number, frozen branch, and the stated requirement
```

| Case | Stage | What's wrong |
|---|---|---|
| 03 Databases | Test | No Postgres service container; readiness check times out |
| 04 Parallelism | Test | `parallelism: 2` is set but the suite never splits |
| 05 Orbs | Build | Empty `orbs:` block; `ruby -v` exits 127 |
| 06 Caching | Build | Cache steps commented out; `yarn install --offline` fails |
| 07 Test Summary | Test | JUnit written but never published |
| 08 Branch Filters | Deploy | Green pipeline; deploy ran on a non-production branch |
| 09 Tag Filters | Deploy | Green pipeline; deploy ran on a branch push, not a beta tag |
| 13 Workspaces | Build | Builds persist the wrong paths; deploy finds no apps |

### `docs/koan-answers-README.md` — the locked answer keys
 
For each case: the requirement from the Koan README, the observed error, the
plausible-but-incorrect fixes, the correct fix, and the contract table every
generated task's `expected-result` must match.
 
**This file is the point of the whole design.** The answers come from
CircleCI's published Training Koans and from pipelines whose fixes were
verified by turning the build green — not from a model's judgment. That is what
lets the three prompt suites be authored by models without the models deciding
what "correct" means.
 
### `docs/koan-task-generation-prompt.md` — how the suites were made
 
The exact instructions each authoring model received. Three rules matter:
 
1. Copy the schema and `expected-result` **verbatim** — authors write the
   question, never the answer.
2. Reproduce the frozen artifacts exactly; never summarize or invent them.
3. State the **requirement**, never the **defect**.
Claude, ChatGPT and Gemini each received this file plus the eight artifact
blocks, in separate sessions, with no shared context. Only the phrasing varied.
 
### `tasks-koans-*.yaml` — the three suites
 
Three files, same eight cases, same locked answers, different wording. Run with
[MindTrial](https://github.com/petmal/MindTrial).
 
Holding the artifact and the answer fixed while varying only the question is
what isolates prompt phrasing as a variable — and phrasing turned out to move
results as much as model choice did.
 
### `results/` — raw scores
 
Per-task CSV and JSONL for every run, split by suite. See
[`results/README.md`](results/README.md) for which file produced which
published figure, and for the runs kept because their *failure mode* is itself
a finding.
 
Columns include status, duration, input and output tokens, and estimated cost
per task, so you can recompute any published number.
 
### `config-koans.yaml` — provider configuration
 
Template with no credentials. Fourteen runs across Anthropic, OpenAI, Google
(Vertex AI via ADC) and AWS Bedrock. The inline comments document the
non-obvious parts — the Bedrock OpenAI-compatible endpoint path, which model
IDs are reachable, and per-provider rate limits.