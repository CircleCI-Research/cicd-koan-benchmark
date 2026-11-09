# Tie rule

How results are reported when models score within run-to-run noise of each
other. Results: [README → Dealing with ties](../README.md#dealing-with-ties).

## Why a rule is needed

Each run is 24 tasks, so one task is worth about 4.17 points of pass rate. Across
five identical runs the top score changed hands, so a one-task difference
between models is inside normal run-to-run variance. A single run cannot tell a
real capability gap from noise.

## The rule

1. **Score:** report mean passes out of 24 and the min–max range across the
   five runs.
2. **Ties:** any model whose range overlaps the top model's range is reported
   as tied. No single winner is named inside a tied group.
3. **Cost:** `$/correct` = total spend across all five runs ÷ total passes
   across all five runs. `$/task` = total spend ÷ 120 tasks.
4. **Speed:** median of every per-task duration across all five runs.
5. **Errors:** errors (timeouts, gateway failures, no response) count against
   pass rate, are excluded from accuracy, and are reported separately.

## Boundary cases

A model that meets rule 2 only through its single best run and has a much
wider spread than the rest of the group is reported as a **boundary case**, not
a peer.

In this study that is Grok 4.6: its range is 18–23, so it overlaps the top range
(23–24) only through run 4. It also has the widest spread in the study and the
only errors (7 of 120).

## Result of applying the rule

- **Top range:** 23–24 (GPT-5.6 Sol, Claude Opus 5.5, Claude Opus 5).
- **Tied group (7):** GPT-5.6 Sol, Claude Opus 5.5, Claude Opus 5, GPT-5.6
  Luna, Claude Fable 5.1, GPT-6 Astra, Gemini 3.1 Pro.
- **Boundary case:** Grok 4.6.
- **Not tied:** every other model; the next-highest ceiling is Claude Sonnet 5
  at 22.
