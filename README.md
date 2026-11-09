# CI/CD Koan Benchmark

A benchmark of how well LLMs triage broken CircleCI jobs.

- **Cases:** 8 frozen CircleCI failures from the public [CircleCI Training Koans](https://github.com/CircleCI-Support/CircleCI-Training-Koans) teaching corpus (snapshots in [`corpus/`](corpus/)), covering Build (koan-05/06/13), Test (koan-03/04/07), and Deploy (koan-08/09).
- **Suites:** each case was written up as a task three times, once each by ChatGPT, Claude, and Gemini (`tasks-koans-*.yaml`), to test how wording affects results.
- **Harness:** runs used [MindTrial](https://github.com/petmal/MindTrial) (MPL-2.0); this repo is the benchmark corpus, tasks, and results — not a fork of the harness.
- **Scoring:** models return a JSON diagnosis; a task passes only if every field matches the locked answer.
- **Results:** raw CSVs are in `results/`, and costs come from `pricing/pricing.go` (MPL-2.0; see [`NOTICE`](NOTICE)).
- **License:** MIT for original CircleCI Research content ([`LICENSE`](LICENSE)). Model display names are the labels used in the evaluation config for each run.

## CI/CD Koan Benchmark Results (October 21–27, 2026)

15 models × 8 frozen CI/CD cases × 3 author suites (ChatGPT, Claude, Gemini) × 5 runs = 1,800 tasks (120 per model). A task passes only if every field of its JSON answer matches the locked answer.

**Source files.** Run *N* means the *N*th run of each suite in chronological order:

| Run | koans-chat | koans-claude | koans-gemini |
|---|---|---|---|
| 1 | [`results-chat-15-27-10.csv`](results/koans-chat/results-chat-15-27-10.csv) | [`results-claude-15-27-40.csv`](results/koans-claude/results-claude-15-27-40.csv) | [`results-gemini-15-54-40.csv`](results/koans-gemini/results-gemini-15-54-40.csv) |
| 2 | [`results-chat-21-24-58.csv`](results/koans-chat/results-chat-21-24-58.csv) | [`results-claude-14-33-46.csv`](results/koans-claude/results-claude-14-33-46.csv) | [`results-gemini-09-25-51.csv`](results/koans-gemini/results-gemini-09-25-51.csv) |
| 3 | [`results-chat-22-30-31.csv`](results/koans-chat/results-chat-22-30-31.csv) | [`results-claude-15-01-02.csv`](results/koans-claude/results-claude-15-01-02.csv) | [`results-gemini-09-40-58.csv`](results/koans-gemini/results-gemini-09-40-58.csv) |
| 4 | [`results-chat-10-24-15.csv`](results/koans-chat/results-chat-10-24-15.csv) | [`results-claude-15-11-09.csv`](results/koans-claude/results-claude-15-11-09.csv) | [`results-gemini-09-52-20.csv`](results/koans-gemini/results-gemini-09-52-20.csv) |
| 5 | [`results-chat-11-34-53.csv`](results/koans-chat/results-chat-11-34-53.csv) | [`results-claude-13-07-18.csv`](results/koans-claude/results-claude-13-07-18.csv) | [`results-gemini-10-11-34.csv`](results/koans-gemini/results-gemini-10-11-34.csv) |

Costs come from the `EstCostUSD` column, which uses `pricing/pricing.go` list prices without cached-input or batch discounts. $/task is total spend ÷ 120, and $/correct is total spend ÷ passes.

### Dealing with Ties

Rule ([full rule](docs/tie-rule.md)): report the mean passes out of 24 and the min–max range across the five runs. Any model whose range overlaps the top model's range (23–24) is reported as tied.

| Model | Provider | Mean /24 | Range | Per-run | $/correct | $/task | Median |
|---|---|---|---|---|---|---|---|
| GPT-5.6 Sol | OpenAI | 23.4 | 23–24 | 23, 23, 23, 24, 24 | $0.01407 | $0.01372 | 4.34s |
| Claude Opus 5.5 | Anthropic | 23.4 | 23–24 | 23, 23, 23, 24, 24 | $0.01881 | $0.01834 | 6.82s |
| Claude Opus 5 | Anthropic | 23.4 | 23–24 | 23, 24, 23, 24, 23 | $0.02461 | $0.02400 | 8.32s |
| GPT-5.6 Luna | OpenAI | 23.2 | 23–24 | 24, 23, 23, 23, 23 | $0.00089 | $0.00086 | 4.15s |
| Claude Fable 5.1 | Anthropic | 23.2 | 23–24 | 24, 23, 23, 23, 23 | $0.04269 | $0.04126 | 7.45s |
| GPT-6 Astra | OpenAI | 23.0 | 23–23 | 23, 23, 23, 23, 23 | $0.03270 | $0.03134 | 11.56s |
| Gemini 3.1 Pro | Google | 22.4 | 22–23 | 23, 22, 22, 23, 22 | $0.00497 | $0.00464 | 11.66s |
| Grok 4.6 | xAI (via Bedrock) | 20.4 | 18–23 | 18, 21, 19, 23, 21 | $0.01397 | $0.01187 | 24.33s |
| Claude Sonnet 5 | Anthropic | 20.2 | 19–22 | 19, 20, 22, 21, 19 | $0.01132 | $0.00953 | 5.58s |
| GPT-5.6 Terra | OpenAI | 20.0 | 19–21 | 20, 19, 20, 21, 20 | $0.00695 | $0.00579 | 3.48s |
| GPT-5 Mini (high) | OpenAI | 19.6 | 19–20 | 20, 20, 20, 19, 19 | $0.00863 | $0.00704 | 27.69s |
| GPT-5 Mini (low) | OpenAI | 17.8 | 16–19 | 17, 16, 19, 18, 19 | $0.00158 | $0.00117 | 4.61s |
| Gemini 2.5 Flash | Google | 16.8 | 15–18 | 15, 17, 18, 17, 17 | $0.00160 | $0.00112 | 7.84s |
| Claude Haiku 4.5 | Anthropic | 15.6 | 14–18 | 16, 14, 18, 14, 16 | $0.00492 | $0.00320 | 5.17s |
| DeepSeek V3.2 | DeepSeek (via Bedrock) | 13.2 | 12–15 | 12, 13, 13, 15, 13 | $0.00254 | $0.00140 | 10.19s |

Grok 4.6 had 7 errors out of 120 tasks (Bedrock TCP read timeouts and gateway 500s): 2 in chat run 1, 3 in chat run 3, and 2 in chat run 5. Every other model had zero errors. Grok's median including errored tasks is 24.33s; excluding them it is 23.54s.

**Tied group (7):** GPT-5.6 Sol, Claude Opus 5.5, Claude Opus 5, GPT-5.6 Luna, Claude Fable 5.1, GPT-6 Astra, Gemini 3.1 Pro. Within this group, $/correct ranges 48× (Luna $0.00089 to Fable $0.04269).

**Boundary case:** Grok 4.6's ceiling of 23 (run 4 only) overlaps the top range, so it meets the rule. It is reported as a boundary case rather than a peer because it has the widest spread in the study (18–23) and the only errors.

### What Stayed vs. Moved

| Stayed | Moved |
|---|---|
| GPT-5.6 Luna inside the tied group every run (24, 23, 23, 23, 23) | Which model led: run 1 Fable 5.1 + Luna (24); run 2 Opus 5 (24); run 3 nobody hit 24 (six-way tie at 23); run 4 Opus 5.5 + Opus 5 + Sol (24); run 5 Opus 5.5 + Sol (24) |
| GPT-6 Astra at exactly 23 in all five runs | Grok 4.6 swinging 18 to 23 |
| The ordering of the three suites on koan-09 (ChatGPT > Claude > Gemini every run) | GPT-5 Mini (low) swinging 16 to 19; Claude Haiku 4.5 swinging 14 to 18 |
| Control (Gemini 2.5 Flash) below the tied group every run (15–18) | Whether anyone reached 24/24 (four of five runs yes, run 3 no) |

### By Stage (top-scoring ties)

Build = koan-05/06/13 and Test = koan-03/04/07 (45 attempts per model each). Deploy = koan-08/09 (30 attempts per model). Rows are sorted by $/correct within each stage.

| Stage | Model | Passes | $/correct | Median |
|---|---|---|---|---|
| Build | Claude Opus 5.5 | 45/45 | $0.01665 | 6.37s |
| | Claude Opus 5 | 45/45 | $0.02129 | 8.00s |
| | GPT-6 Astra | 45/45 | $0.02995 | 11.80s |
| | Claude Fable 5.1 | 45/45 | $0.04023 | 7.56s |
| Test | GPT-5.6 Luna | 45/45 | $0.00076 | 3.45s |
| | Gemini 3.1 Pro | 45/45 | $0.00512 | 10.45s |
| | GPT-5.6 Sol | 45/45 | $0.01183 | 4.05s |
| | Claude Opus 5.5 | 45/45 | $0.01866 | 6.96s |
| | Claude Opus 5 | 45/45 | $0.02380 | 8.25s |
| | GPT-6 Astra | 45/45 | $0.03168 | 10.92s |
| | Claude Fable 5.1 | 45/45 | $0.03878 | 7.29s |
| Deploy | GPT-5.6 Luna | 28/30 | $0.00151 | 6.76s |
| | GPT-5.6 Sol | 28/30 | $0.02219 | 6.97s |
| | Claude Opus 5.5 | 27/30 | $0.02267 | 7.94s |
| | Claude Opus 5 | 27/30 | $0.03151 | 10.25s |

Nobody was perfect on Deploy. GPT-5.6 Luna matched GPT-5.6 Sol's 28/30 at about 1/15 of Sol's cost per correct answer.

### koan-09: Wording and Its Effects

Locked answer: `fix_target_job: all_jobs`. 118 of 225 attempts passed. Of the 107 non-passes, 102 answered `deploy` (the koan-08 answer), 3 named the right target but got a boolean wrong, and 2 were Grok errors.

| Suite Author | What the Task States | Mean Passed (of 15) | Passes per Run |
|---|---|---|---|
| ChatGPT | "the build, test, and deploy jobs must participate in that tagged flow… Determine the affected workflow scope…" | 11.4 | 11, 12, 12, 11, 11 |
| Claude | "the deploy job must run only for tagged commits… which job(s) need their workflow configuration changed?" | 9.2 | 9, 8, 10, 10, 9 |
| Gemini | "we only ever want to run our deploy job when we commit a tag… Ensure the deploy job only runs for tagged commits…" | 3.0 | 2, 3, 3, 3, 4 |

### koan-09: Which Models Resisted

| Model | koan-09 passes (of 15) | ChatGPT / Claude / Gemini suite (of 5 each) |
|---|---|---|
| GPT-5.6 Sol | 13 | 5 / 5 / 3 |
| GPT-5.6 Luna | 13 | 4 / 5 / 4 |
| Claude Opus 5.5 | 12 | 5 / 5 / 2 |
| Claude Opus 5 | 12 | 5 / 5 / 2 |
| Claude Fable 5.1 | 11 | 5 / 5 / 1 |
| Gemini 3.1 Pro | 10 | 5 / 5 / 0 |
| GPT-6 Astra | 10 | 5 / 5 / 0 |
| GPT-5.6 Terra | 10 | 4 / 4 / 2 |
| Claude Sonnet 5 | 8 | 5 / 2 / 1 |
| Grok 4.6 | 8 | 3 / 5 / 0 |
| Claude Haiku 4.5 | 4 | 4 / 0 / 0 |
| GPT-5 Mini (low) | 4 | 4 / 0 / 0 |
| GPT-5 Mini (high) | 2 | 2 / 0 / 0 |
| Gemini 2.5 Flash | 1 | 1 / 0 / 0 |
| DeepSeek V3.2 | 0 | 0 / 0 / 0 |

### TL;DR

| Priority | Pick | Why |
|---|---|---|
| Best value | GPT-5.6 Luna | In the tied group all five runs at $0.00089 per correct answer, 48× cheaper than the most expensive tied model (Claude Fable 5.1, $0.04269) |
| Best accuracy | A seven-way tie | GPT-5.6 Sol, Claude Opus 5.5, Claude Opus 5, GPT-5.6 Luna, Claude Fable 5.1, GPT-6 Astra, Gemini 3.1 Pro: means 22.4 to 23.4 out of 24, all ranges overlapping |
| Fastest | GPT-5.6 Terra | 3.48s median, but it averaged 20 of 24 correct, below the tied group. The fastest tied model is GPT-5.6 Luna at 4.15s. |

Per 1,000 correct diagnoses at these rates: GPT-5.6 Luna about $0.89, Claude Opus 5.5 about $18.81, Claude Fable 5.1 about $42.69.
