# Koan answer keys (locked)

Human-locked from the frozen configs, logs, and Koan READMEs in this `corpus/`
folder. `expected-result` in any generated
[MindTrial](https://github.com/petmal/MindTrial) task must match **Correct
Fix** / the contract table below. A model may draft prompt wrapping only. It must not invent these keys.

Ground truth is the frozen branch, not the public Koan README, when the two
disagree.

## koans-03-databases 

**Requirement (Koan README):** The user is attempting to run a node web application which connects to a Postgresql database. The user wants to know how to add the database to their project.

**Goals:** Modify the included config file to get a green job. Share link to a green job.

**Error:** `dockerize -wait tcp://localhost:5432 -timeout 1m` retried for 60s with `connection refused` on every attempt, then `Timeout after 1m0s waiting on dependencies to become available: [tcp://localhost:5432]`. Exit 1.

Step list shows `install dockerize` SUCCESS, so the tool itself works. Nothing is listening on port 5432. The job's `docker:` list has a single image (`circleci/node`) and no Postgres service container.

**Incorrect Fix:**
- Increase the dockerize timeout. Nothing will ever bind 5432.
- Install Postgres with `apt-get` in the primary container. That is not how CircleCI service containers work, and it is not the Koan.
- Remove the `Wait for db` step. The job can go green while the app still has no database.
- Point dockerize at a different host or port.
- Use a remote hosted database instead of a service container.

**Correct Fix:** Add a Postgres image as a second entry under the job's `docker:` list (a service container) so something listens on `localhost:5432`.

**Question scope (koans-03):** Why does nothing respond on port 5432, and what
config change provides the database?

## koans-04-parallelism 

**Requirement (Koan README):** Run the test scripts in parallel without modifying
the workflow. Get a green build.

**Goals:** Split `src/test_hello.sh` and `src/test_goodbye.sh` across parallel
nodes. Green job.

**Frozen vs public:** The public Teaching Koan starts with two sequential `run`
steps and no `parallelism`. This freeze already sets `parallelism: 2` but still
runs every `src/test_*.sh` on every node. A split list is written to
`/tmp/split.txt` and printed, but the run loop iterates `src/test_*.sh` instead
of that list. Both parallel containers exit 1 after running 2 files while the
split list had 1.

**Incorrect Fix:**
- Set a higher `parallelism` value only. Without splitting, every node still runs
  the full suite.
- Run the two scripts as separate workflow jobs. The requirement is parallelism
  within the job, without modifying the workflow shape.
- Drop `parallelism` and keep sequential runs. Does not meet the goal.
- Rely on the `/tmp/split.txt` line already in the config — it is not wired into
  what actually executes.
- Conclude the job is fine because each individual script echoed successfully.

**Correct Fix:** Feed `circleci tests glob "src/test_*.sh" | circleci tests split`
into the commands that actually run so each node executes one shard (typically
via `xargs` / a loop over the split list).

**Question scope (koans-04):** Was the suite split across nodes, and what change
makes each parallel container run only its shard?

## koans-05-orbs 

**Requirement (Koan README):** Add the Ruby Orb (`circleci/ruby`) to the config file and install Ruby version `2.6.3` with the orb. The build should be green if Ruby is installed properly. The README states the chosen image does not have Ruby.

**Goals:** Add the Ruby Orb to the config file. Install Ruby version 2.6.3 with the orb. Build should be green if Ruby is installed properly.

**Error:** `ruby -v` → `/bin/bash: ruby: command not found`, exit 127.

Step list shows only four steps: environment, env vars, checkout, `ruby -v`. No step between checkout and `ruby -v`, so nothing attempted to install Ruby. The `orbs:` block contains only the comment `# Add Orb here`.

**Incorrect Fix:**
- Change the base image from `cimg/base:stable` to a Ruby image. Would produce a green build but does not satisfy the requirement to use the orb.
- Install Ruby with `apt-get` in a `run` step. Same problem.
- Install a Ruby version other than `2.6.3`.
- Remove or replace the `ruby -v` step.

**Correct Fix:** Declare `circleci/ruby` in the `orbs:` block and add the orb's install step, pinned to version `2.6.3`, before `ruby -v`.

**Question scope (koans-05):** Did any step attempt to install Ruby, and what
change satisfies the orb requirement?


## koans-06-caching 

**Requirement (Koan README):** Store a cache of `package.json` so it can be restored in subsequent builds, and know what to do to force a new save of the cache. Specifically: uncomment the `save_cache` and `restore_cache` steps, update the `key` of each to point at the proper file, confirm a save then a restore, then edit the config to bust the cache so it stores a new one.

**Note on this case:** the frozen branch differs from the public Koan in two ways. Image is `cimg/node:current` (public: `cimg/node:12.16`), and the install command is `yarn install --offline` (public: `yarn install --immutable`). `--offline` is deliberate on the frozen branch to keep the case red. Ground truth follows the frozen branch. There is no `yarn.lock` in this repo (`info No lockfile found` in the log).

**Error:** `yarn install --offline` fails: `Couldn't find any versions for "left-pad" that matches "1.3.0" in our cache`. Exit 1. `restore_cache` and `save_cache` are commented out, so no cache is present to satisfy an offline install.

**Incorrect Fix:**
- Remove `--offline` or switch to `--immutable`. Produces a green job by allowing a network fetch, which defeats the caching requirement.
- Key the cache on `{{ checksum "yarn.lock" }}`. Conventional for yarn, but **no `yarn.lock` exists** — the checksum would fail or resolve to nothing. The README names `package.json`.
- Uncomment only `restore_cache` without `save_cache`. Nothing is ever stored, so every run misses.
- Uncomment only `save_cache` without `restore_cache`. Nothing is ever restored.
- Change the cache key to bust it without first establishing a working save/restore.

**Correct Fix:** Uncomment both `restore_cache` and `save_cache`, and set each key to a checksum of `package.json`. To bust the cache afterward, change the version prefix in the key (e.g. `yarn-packages-v1-` → `yarn-packages-v2-`).

**Question scope (koans-06):** Why does an offline install fail with no cache,
and what must be enabled and keyed to which file?

## koans-07-test-summary 

**Requirement (Koan README):** Congrats, your tests have passed! Your test results were exported to `~/project/results/test_results.xml`. Display the test results in the test summary tab, and export the file as an artifact.

**Goals:** Get a green build. Have test results displayed in the test summary tab. Export the test results to an artifact. Share link to a green job.

**Frozen vs public (lock to frozen):** The public README says tests already passed and implies a green job missing only publish steps. The frozen branch writes a passing JUnit suite to `~/project/results/test_results.xml` and then `exit 1`. Step `Run tests` FAILED. The job is **red**. There is no `build-tools` orb and no `upload:` flag in this freeze.

**Error:** JUnit XML is written (`Wrote ~/project/results/test_results.xml`) but never published. Neither `store_test_results` nor `store_artifacts` is present, so the test summary tab shows no results and no artifact is stored. The step then exits 1.

**Incorrect Fix:**
- Conclude the test suite did not run. The log shows the results file was produced.
- Add `store_artifacts` only. Satisfies the artifact goal; test summary stays empty.
- Add `store_test_results` only. Satisfies the test summary goal; no artifact.
- Remove `exit 1` only. Can make the job green without publishing results.
- Invent an `upload: true` flag or a `build-tools/test-results` orb step. Those are not in this frozen config.

**Correct Fix:** Add both `store_test_results` and `store_artifacts`, each pointing at `~/project/results`. (A live green repair also has to stop exiting 1. That is freeze scaffolding, not the Koan lesson — do not score it as the `fix`.)

**Question scope (koans-07):** Ask what change is required for results to appear
in the test summary tab and as an artifact. Do NOT ask what makes the job green —
`exit 1` is freeze scaffolding, and a model naming it is answering a different
question.

## koans-08-branch-filters 

**Requirement (Koan README):** A workflow contains a `deploy` job. The `deploy` job should only ever run when committing to the `production` branch.

**Goals:** Ensure the deploy job only runs for the production branch. Create the required branch and meet the requirements above for a green Workflow. Share link to a green Workflow. 

Overall: pipeline GREEN, behavior WRONG. The deploy job has no branch filter, so it executes on every branch.

**Error:** Pipeline #151 is **green** on branch `koans-08-branch-filters` — not `production` — and the `deploy` job ran anyway (SUCCESS, 21s). There is no failing step and no error output. The defect is behavioural: the workflow declares `build`, `test-production`, and `deploy` chained by `requires`, with no `filters` block on any job, so `deploy` executes on every branch.

**Incorrect Fix:**
- Conclude nothing is wrong. The pipeline succeeded, so a scan for error text finds nothing.
- Put the branch filter on `build` or `test-production` instead of `deploy`. The job name `test-production` is a distractor.
- Use `filters: branches: ignore:` with a list of non-production branches. Fails for any branch not enumerated.
- Place `filters` inside the job definition under `jobs:` rather than under the job's entry in the workflow. Filters belong in the workflow.
- Rely on `requires` ordering to gate deployment. `requires` controls order, not which branches run.

**Correct Fix:** Add `filters: branches: only: production` to the `deploy` job's entry in the `build-test-and-deploy` workflow.

**Question scope (koans-08):** Is the observed behaviour correct for this branch,
and what change restricts deploy to production?

## koans-09-tag-filters 

**Requirement (Koan README):** `deploy` should only run for beta product tags
(e.g. `beta-main-0.1.1`, `beta-secondary-0.1.1`), never for `alpha-*` tags.

**Goals:** Gate deploy to matching tags. Green workflow when the tag is correct.

Overall: pipeline GREEN, behavior WRONG.
The deploy job has no tag filters, so it executes on branch pushes.
Requirement: deploy only for tags matching /^beta-.*/ (e.g. beta-main-0.1.1).

**Error:** Pipeline #156 is **green** on branch `koans-09-tag-filters` — not a
tag — and `deploy` ran (SUCCESS). No failing step. No `filters` on any workflow
job. Same behavioural shape as koans-08, but the policy is tag-based (`/^beta-.*/`),
not branch-based.

**Incorrect Fix:**
- Conclude nothing is wrong. The pipeline succeeded.
- Copy the koans-08 branch filter (`branches: only: production`). Wrong policy.
- Use `tags: only: /.*/` or otherwise allow `alpha-*` tags.
- Put tag filters **only on `deploy`**. Stops branch-push deploys, but on a
  `beta-*` tag push `build`/`test` never run (no tag filters), so the
  `requires` chain never reaches deploy — workflow stays incomplete.
- Put tag filters only on `build` / `test` and leave `deploy` ungated.
- Place `filters` under the job definition in `jobs:` instead of the workflow entry.
- Rely on `requires` to restrict tags. `requires` is order, not tag gating.

**Correct Fix:** Add compatible workflow `filters` on **every** job in the
`requires` chain (`build`, `test`, and `deploy`) so tags match `/^beta-.*/`.
On `deploy`, also ignore branches (`branches: ignore: /.*/`) so ordinary
branch pushes do not deploy. Tag pushes do not run a job unless that job has
an explicit tag filter — filtering `deploy` alone is not enough.

**Question scope (koans-09):** Is the observed behaviour correct for this branch
push, and which jobs need tag filters so deploy both stays off branches and
still runs on `beta-*` tags?

## koans-13-workspaces

**Requirement (Koan README):** Persist each platform app from the build jobs into
the final `deploy` job, then upload all apps as artifacts.

**Goals:** `app/linux/app.linux`, `app/windows/app.windows`, and `app/mac/app.mac`
available in deploy; stored as artifacts.

**Error:** Pipeline #167 — build jobs SUCCESS; deploy FAILED. Deploy attaches the
workspace successfully, but contents are only `meta/linux`, `meta/mac`, and
`meta/windows`. `Verify apps present` fails because `app/...` files were never
persisted. Each build generates the real app then persists `meta/<platform>/`
instead.

**Incorrect Fix:**
- Add `store_artifacts` on each build job and skip workspaces. Artifacts are not
  how downstream jobs receive files in this Koan.
- Fix only one platform's `persist_to_workspace` paths.
- Change `attach_workspace.at` without persisting the real `app/` paths.
- Remove the verify step so deploy goes green without the apps.
- Persist additional `meta/` files. Metadata is not what deploy verifies.

**Correct Fix:** In each build job, `persist_to_workspace` the real generated app
path (e.g. `app/linux`, `app/windows`, `app/mac` with an appropriate `root`), keep
`attach_workspace` in deploy, then `store_artifacts` of `app`.

**Question scope (koans-13):** Why are the apps missing in deploy despite green
builds, and what must be persisted?

---

## Answer contracts for task generation

Each generated task's `expected-result` must match the row below. Distractors listed
under **Incorrect Fix** should appear in the task's option set where the response
format is an enum — a task whose options contain no plausible wrong answer will not
discriminate.

| Case | Field | Locked value |
|---|---|---|
| koans-03 | `root_cause` | `no_postgres_service_container` |
| koans-03 | `fix` | `add_postgres_service_image` |
| koans-05 | `install_attempted` | `false` |
| koans-05 | `fix` | `add_ruby_orb` |
| koans-06 | `cache_restored` | `false` |
| koans-06 | `checksum_target` | `package.json` |
| koans-06 | `fix` | `enable_restore_and_save_cache` |
| koans-07 | `results_produced` | `true` |
| koans-07 | `job_green` | `false` |
| koans-07 | `missing_store_test_results` | `true` |
| koans-07 | `missing_store_artifacts` | `true` |
| koans-07 | `fix` | `add_store_test_results_and_store_artifacts` |
| koans-08 | `pipeline_green` | `true` |
| koans-08 | `behaviour_correct` | `false` |
| koans-08 | `fix_target_job` | `deploy` |
| koans-08 | `fix` | `filters_branches_only_production` |
| koans-04 | `suite_split` | `false` |
| koans-04 | `each_node_ran_full_suite` | `true` |
| koans-04 | `fix` | `use_circleci_tests_split` |
| koans-09 | `pipeline_green` | `true` |
| koans-09 | `behaviour_correct` | `false` |
| koans-09 | `fix_target_job` | `all_jobs` |
| koans-09 | `fix` | `filters_tags_only_beta` |
| koans-13 | `builds_green` | `true` |
| koans-13 | `deploy_green` | `false` |
| koans-13 | `workspace_had_apps` | `false` |
| koans-13 | `fix` | `persist_app_paths` |

## Verification checklist for every generated task

1. Does `expected-result` match the locked value for that case?
2. Is the answer derivable from the artifacts included in the prompt?
3. Does the option set contain at least one distractor from **Incorrect Fix**?
4. Does the prompt avoid naming the fix in its own wording?
