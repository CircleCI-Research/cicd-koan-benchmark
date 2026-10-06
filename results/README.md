# Raw results

Each run is one suite against one or more models. The published tables
pool all three suites.

| File | Suite | Models |
|---|---|---|
| results-claude-10-30-37 | Claude-authored | 11 (DeepSeek on json-schema — see note) |
| results-claude-10-48-21 | Claude-authored | DeepSeek V3.2, Grok 4.6 (json-object rerun) |
| results-claude-14-39-23 | Claude-authored | Claude Opus 5.5 |
| results-claude-15-13-10 | Claude-authored | Claude Fable 5.1, GPT-6 Astra |
| ... |

The published figures use the json-object DeepSeek rerun. The original
json-schema run is kept because the failure mode is itself a finding:
exactly 16,384 output tokens with no stop token on koan09.