# GitHub Workflow Test

A sandbox repository for experimenting with GitHub Actions: runners, job
dependencies, environment variables, and artifacts. Every workflow is triggered
manually with `workflow_dispatch`, so nothing runs on push or pull request.

## Workflows

| Workflow | File | Demonstrates |
|----------|------|--------------|
| Different Runners | [`different-runners.yml`](.github/workflows/different-runners.yml) | Running jobs on Ubuntu, Windows, and macOS runners, and inside an Alpine container |
| Job Dependencies: Basic | [`job-dependencies-basic.yml`](.github/workflows/job-dependencies-basic.yml) | A chained job plus a final job that joins two branches |
| Job Dependencies: With Independent Jobs | [`job-dependencies-with-independent-jobs.yml`](.github/workflows/job-dependencies-with-independent-jobs.yml) | A dependency chain alongside a job with no `needs:` |
| Job Dependencies: Fan-Out | [`job-dependencies-fan-out.yml`](.github/workflows/job-dependencies-fan-out.yml) | One job feeding multiple downstream jobs |
| Job Dependencies: Parallel Chains | [`job-dependencies-parallel-chains.yml`](.github/workflows/job-dependencies-parallel-chains.yml) | Two independent dependency chains running in parallel |
| Job Dependencies: Fan-In | [`job-dependencies-fan-in.yml`](.github/workflows/job-dependencies-fan-in.yml) | Multiple jobs converging on a single downstream job |
| Job Dependencies: Fan-In With Final Join | [`job-dependencies-fan-in-with-final-join.yml`](.github/workflows/job-dependencies-fan-in-with-final-join.yml) | A fan-in followed by a final join |
| Variables in Environments | [`variables-in-environments.yml`](.github/workflows/variables-in-environments.yml) | Reading the same repository variable from `dev`, `prod`, and global scope |
| Persisting Data with Artifacts | [`persisting-data-with-artifacts.yml`](.github/workflows/persisting-data-with-artifacts.yml) | Passing a file from one job to another with `upload-artifact` and `download-artifact` |

## Job dependency shapes

The six `job-dependencies-*` workflows show how `needs:` controls execution
order. Jobs without `needs:` start immediately; jobs with `needs:` wait for all
listed jobs to succeed.

```text
Basic
  root-job-2 -> chained-job -> final-join-job
  root-job-1 ----------------> final-join-job

With Independent Jobs
  root-job-2 -> chained-job -> join-job
  root-job-1 ----------------> join-job
  independent-job (starts immediately)

Fan-Out
  root-job-2 -> chained-job -> join-job
  root-job-1 ----------------> join-job
                 chained-job -> downstream-job

Parallel Chains
  root-job-2 -> first-chain-job -> third-chain-job
  root-job-1 -> second-chain-job

Fan-In
  root-job-1 -\
               +-> fan-in-job -> downstream-job
  root-job-2 -/
  root-job-1 -> single-dependency-job

Fan-In With Final Join
  root-job-1 -\
               +-> fan-in-job -> downstream-job -\
  root-job-2 -/                                     +-> final-join-job
  root-job-1 --------------------------------------/
```

## Running a workflow

From the GitHub UI: open the **Actions** tab, select a workflow in the left
sidebar, then choose **Run workflow**.

From the command line with the [GitHub CLI](https://cli.github.com/):

```bash
gh workflow run different-runners.yml
gh workflow list
gh run watch
```

## Configuration

`variables-in-environments.yml` reads `vars.TEST_VAR`. That variable is not
stored in this repository; define it under **Settings → Secrets and variables →
Actions**:

- **Repository variable** `TEST_VAR` — used by `show-global-variable`.
- **Environment variables** `TEST_VAR` for the `dev` and `prod` environments —
  used by the environment-scoped jobs.

Define the `dev` and `prod` environments under **Settings → Environments**. If
the variable is missing, the workflow still runs but prints an empty value.

`persisting-data-with-artifacts.yml` uploads `persisted-data.txt` as the
`persisted-data` artifact and downloads it in a dependent job. Artifacts
uploaded by a workflow run are available on the run summary page.

## Requirements

- GitHub-hosted runners (`ubuntu-24.04`, `windows-2022`, `macos-14`).
- No application code, build step, or local dependencies.
