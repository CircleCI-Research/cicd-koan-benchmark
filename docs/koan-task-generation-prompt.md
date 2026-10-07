# Generate Koan tasks without inventing the answer key

Paste this into a **new** Claude, ChatGPT, or Gemini chat. Do not use a
thread that already saw these Koans. The point of three authors is that
prompt wrapping varies; the scorecard must not.

You are drafting CI/CD tasks. You do not choose the answers.

Rules:

1. Copy each `response-result-format` and `expected-result` block below
   **verbatim**. Do not rename fields, add fields, drop fields, or change
   enum values or expected values.
2. You may rewrite the `prompt` only. The prompt must include the frozen
   config and the frozen log/step list for that Koan. The answer must be
   derivable from those artifacts.
3. Do not name the correct `fix` enum value in the prompt prose. Enum
   options will appear in the schema; that is fine.
4. Put at least one Incorrect Fix distractor in the enum — they are
   already there; do not remove them.
5. Output YAML tasks only, same `name` slugs as below (`test/koan03/...`
   etc).
6. If you are unsure of an answer, leave `expected-result` as given.
   Never "improve" it.

## Additional Rules
7. The frozen artifacts are pasted after this prompt, one block ber Koan, labelled by case. Reproduce them EXACTLY in your prompt — same lines, same order, same count. Do not summarize, abbreviate, reformat, or substitute. If any artifact appears missing or unreadable, STOP and say so. Never generate placeholder or plausible contents.

8. State the REQUIREMENT, never the DEFECT. The   
  requirement is what the pipeline was supposed to do; the model needs it. The defect is what is actually wrong; the model must infer it. Do not describe the root cause,name the missing config key, or hint at which field is misconfigured.

Write eight tasks, one per Koan.

## Question Scope for 3 Cases
koan04 — What is `parallelism` set to, and did the suite actually split across nodes? Parallelism IS configured; a prompt that invites "parallelism isn't set" is wrong-scoped.

koan06 - The --offline flag is fixed and must not be changed. Ask which cache steps are missing and which file the key should checksum, given that constraint.
      
koan07 — What change makes results appear in the test summary tab AND as an artifact? Do NOT ask what makes the job green.

## Locked schemas (do not edit)

### koans-03

```yaml
- name: "test/koan03/triage - wait for db connection refused"
  prompt: |
    (your wrapping; include config + log from corpus/koans-03-databases/)
  response-result-format:
    type: object
    additionalProperties: false
    properties:
      root_cause:
        type: string
        enum:
          - no_postgres_service_container
          - dockerize_not_installed
          - wait_timeout_too_short
          - wrong_wait_port
          - node_image_missing_postgres_client
      fix:
        type: string
        enum:
          - add_postgres_service_image
          - increase_timeout
          - install_postgres_via_apt
          - remove_wait_step
          - change_wait_host
    required: ["root_cause", "fix"]
  expected-result:
    - root_cause: no_postgres_service_container
      fix: add_postgres_service_image
```

### koans-05

```yaml
- name: "build/koan05/triage - ruby missing after checkout"
  prompt: |
    (your wrapping; include config + log from corpus/koans-05-orbs/)
  response-result-format:
    type: object
    additionalProperties: false
    properties:
      install_attempted: { type: boolean }
      fix:
        type: string
        enum:
          - add_ruby_orb
          - change_base_image
          - install_via_apt
          - increase_timeout
    required: ["install_attempted", "fix"]
  expected-result:
    - install_attempted: false
      fix: add_ruby_orb
```

### koans-06

```yaml
- name: "build/koan06/triage - yarn offline cache miss"
  prompt: |
    (your wrapping; include config + log from corpus/koans-06-caching/)
  response-result-format:
    type: object
    additionalProperties: false
    properties:
      cache_restored: { type: boolean }
      checksum_target:
        type: string
        enum:
          - package.json
          - yarn.lock
          - package-lock.json
          - .cache/yarn
      fix:
        type: string
        enum:
          - enable_restore_and_save_cache
          - drop_offline_flag
          - checksum_yarn_lock
          - restore_only
          - save_only
    required: ["cache_restored", "checksum_target", "fix"]
  expected-result:
    - cache_restored: false
      checksum_target: package.json
      fix: enable_restore_and_save_cache
```

### koans-07

Lock to the frozen branch: the job is red (`exit 1`). There is no
`build-tools` orb. Do not write a prompt that claims the job is green.

```yaml
- name: "test/koan07/triage - junit written not published"
  prompt: |
    (your wrapping; include config + log from corpus/koans-07-test-summary/)
  response-result-format:
    type: object
    additionalProperties: false
    properties:
      results_produced: { type: boolean }
      job_green: { type: boolean }
      missing_store_test_results: { type: boolean }
      missing_store_artifacts: { type: boolean }
      fix:
        type: string
        enum:
          - add_store_test_results_and_store_artifacts
          - set_upload_true
          - store_artifacts_only
          - store_test_results_only
          - remove_exit_1_only
    required:
      - results_produced
      - job_green
      - missing_store_test_results
      - missing_store_artifacts
      - fix
  expected-result:
    - results_produced: true
      job_green: false
      missing_store_test_results: true
      missing_store_artifacts: true
      fix: add_store_test_results_and_store_artifacts
```

### koans-08

```yaml
- name: "deploy/koan08/triage - deploy ran off production"
  prompt: |
    (your wrapping; include config + workflow.txt from corpus/koans-08-branch-filters/)
  response-result-format:
    type: object
    additionalProperties: false
    properties:
      pipeline_green: { type: boolean }
      behaviour_correct: { type: boolean }
      fix_target_job:
        type: string
        enum:
          - deploy
          - build
          - test-production
          - all_jobs
      fix:
        type: string
        enum:
          - filters_branches_only_production
          - filters_branches_ignore_non_production
          - filters_on_job_definition
          - rely_on_requires
          - nothing_wrong
    required:
      - pipeline_green
      - behaviour_correct
      - fix_target_job
      - fix
  expected-result:
    - pipeline_green: true
      behaviour_correct: false
      fix_target_job: deploy
      fix: filters_branches_only_production
```

### koans-04

```yaml
- name: "test/koan04/triage - parallelism without split"
  prompt: |
    (your wrapping; include config + log from corpus/koans-04-parallelism/)
  response-result-format:
    type: object
    additionalProperties: false
    properties:
      suite_split: { type: boolean }
      each_node_ran_full_suite: { type: boolean }
      fix:
        type: string
        enum:
          - use_circleci_tests_split
          - increase_parallelism_only
          - split_into_workflow_jobs
          - drop_parallelism
          - nothing_wrong
    required: ["suite_split", "each_node_ran_full_suite", "fix"]
  expected-result:
    - suite_split: false
      each_node_ran_full_suite: true
      fix: use_circleci_tests_split
```

### koans-09

Lock to the full Koan requirement (green workflow on a `beta-*` tag), not only
stopping the observed branch-push deploy. Tag pushes skip jobs without explicit
tag filters, so a `requires` chain needs filters on **all** jobs — `deploy` alone
is a distractor (`fix_target_job: deploy`).

```yaml
- name: "deploy/koan09/triage - deploy ran without beta tag"
  prompt: |
    (your wrapping; include config + workflow.txt from corpus/koans-09-tag-filters/)
  response-result-format:
    type: object
    additionalProperties: false
    properties:
      pipeline_green: { type: boolean }
      behaviour_correct: { type: boolean }
      fix_target_job:
        type: string
        enum:
          - deploy
          - build
          - test
          - all_jobs
      fix:
        type: string
        enum:
          - filters_tags_only_beta
          - filters_branches_only_production
          - filters_tags_allow_all
          - filters_on_job_definition
          - rely_on_requires
          - nothing_wrong
    required:
      - pipeline_green
      - behaviour_correct
      - fix_target_job
      - fix
  expected-result:
    - pipeline_green: true
      behaviour_correct: false
      fix_target_job: all_jobs
      fix: filters_tags_only_beta
```

### koans-13

```yaml
- name: "build/koan13/triage - workspace missing apps"
  prompt: |
    (your wrapping; include config + log/workflow from corpus/koans-13-workspaces/)
  response-result-format:
    type: object
    additionalProperties: false
    properties:
      builds_green: { type: boolean }
      deploy_green: { type: boolean }
      workspace_had_apps: { type: boolean }
      fix:
        type: string
        enum:
          - persist_app_paths
          - store_artifacts_on_builds_only
          - fix_one_platform_only
          - change_attach_only
          - remove_verify_only
    required:
      - builds_green
      - deploy_green
      - workspace_had_apps
      - fix
  expected-result:
    - builds_green: true
      deploy_green: false
      workspace_had_apps: false
      fix: persist_app_paths
```

After generation, run the checklist in `koan-answers-README.md` and
compare `expected-result` to this file. If they differ, discard the task.
