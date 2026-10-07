## Koans Expected Results

Every generated task's `expected-result` must match the values below, or the
task is discarded. These keys are human-locked from the public Koan READMEs
and the frozen artifacts — no model contributed to them.

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
`requires` chain and nothing runs, so the answer is `all_jobs`. Every failure
on 09 across all 14 models chose `deploy`.