# gha-workflows
| | |
| --- | --- |
| CI | [![CodeQL analysis](https://github.com/UBC-MOAD/gha-workflows/actions/workflows/codeql-analysis-this-repo.yaml/badge.svg)](https://github.com/UBC-MOAD/gha-workflows/actions?query=workflow:CodeQL) |
| Python | [![Python Version](https://img.shields.io/badge/Python-3.13-blue?logo=python&label=Python&logoColor=gold)](https://docs.python.org/3/) |
| Issue Tracker | [![Issue Tracker](https://img.shields.io/github/issues/UBC-MOAD/gha-workflows?logo=github)](https://github.com/UBC-MOAD/Reshapr/issues) |
| Meta | [![Licensed under the Apache License, Version 2.0](https://img.shields.io/badge/license-Apache%202-cb2533.svg)](https://www.apache.org/licenses/LICENSE-2.0) [![Git on GitHub](https://img.shields.io/badge/version%20control-git-blue.svg?logo=github)](https://github.com/UBC-MOAD/gha-workflows) [![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/pre-commit/pre-commit) [![The uncompromising Python code formatter](https://img.shields.io/badge/code%20style-black-000000.svg)](https://black.readthedocs.io/en/stable/) |


Reusable GitHub Actions workflows for MOAD repositories and workflow management tools


## Changes

### 2-Feb-2024

Added `auto-milestone-issue-pr.yaml` workflow to automatically add current milestone to new issues
and PRs.


### 22-Mar-2024

Added Codecov token to `pytest-with-coverage` workflow to re-enable coverage
report comments in pull requests.
Tokens became required for that functionality with Codecov's change to v4.0.0
of its GitHub action.

ref: https://about.codecov.io/blog/january-product-update-updating-the-codecov-ci-uploaders-to-the-codecov-cli/


### 19-Jan-2023

Added `gha_workflows_checker.py` utility script from https://github.com/UBC-MOAD/gha-workflows-checker.


### 2-Dec-2022

Changed to rely on the Slack github app workflows subscription feature to send workflow status
notifications to Slack instead of the 8398a7/action-slack action.

Enable that feature with:

`/github subscribe org/repo workflows:{event:"pull_request","push" branch:"main"}`

ref: https://github.com/integrations/slack#actions-workflow-notifications


## How to Use the Workflows

YAML blobs to use the reusable workflows in other repositories.

### `auto-assign`

```yaml
name: Assign Issue/PR

on:
  issues:
    types:
      - reopened
      - opened
  pull_request:
    types:
      - reopened
      - opened

jobs:
  auto_assign:
    permissions:
      issues: write
      pull-requests: write
    uses: UBC-MOAD/gha-workflows/.github/workflows/auto-assign.yaml@main
```


### `auto-milestone-issue-pr`

```yaml
name: Add Milestone to Issue/PR

on:
  issues:
    types:
      - opened
  pull_request:
    types:
      - opened
    branches:
      - main

jobs:
  add_milestone:
    permissions:
      issues: write
      pull-requests: write
    uses: UBC-MOAD/gha-workflows/.github/workflows/auto-milestone-issue-pr.yaml@main
```


### `codeql-analysis`

**Note:** Each repo should have a different cron schedule.

```yaml
name: "CodeQL"

on:
  push:
    branches: [ '*' ]
  schedule:
    - cron: '20 17 * * 1'

jobs:
  analyze:
    name: Analyze
    permissions:
      actions: read
      contents: read
      security-events: write
    strategy:
      fail-fast: false
      matrix:
        language: [ 'python' ]
    uses: UBC-MOAD/gha-workflows/.github/workflows/codeql-analysis.yaml@SHA
    with:
      language: ${{ matrix.language }}
```

### `pytest-with-coverage`

**Notes:**

* A Codecov token is required for coverage results to be uploaded to
  Codecov and coverage report comments to appear in pull requests.
  Codecov tokens are generated as global upload tokens for organizations
  on Codecov and stored as organization secrets named CODECOV_TOKEN on GitHub.
* Be sure to set the `conda-env-name:` value correctly.

```yaml
name: pytest-with-coverage

on:
  push:
    branches: [ '*' ]

jobs:
  pytest-with-coverage:
    permissions:
      contents: read
      pull-requests: write
    strategy:
      fail-fast: false
      matrix:
        python-version: [ '3.10', '3.11' ]
    uses: UBC-MOAD/gha-workflows/.github/workflows/pytest-with-coverage.yaml@main
    with:
      python-version: ${{ matrix.python-version }}
      conda-env-file: envs/environment-test.yaml
      conda-env-name: <test-env-name>
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```


### `sphinx-linkcheck`

**Notes:**

* Each repo should have a different cron schedule.
  Please see https://salishseacast.slack.com/archives/C01GYJBSF0X/p1608574921004500
* Be sure to set the `conda-env-name:` value correctly

```yaml
name: sphinx-linkcheck

on:
  push:
    branches: [ '*' ]
  schedule:
    - cron: 43 10 13 * *  # 10:43 UTC on the 4th day of each month

jobs:
  sphinx-linkcheck:
    permissions:
      contents: read
    strategy:
      fail-fast: false
      matrix:
        # Need to specify Python version here because we use test env which gets its
        # Python version via matrix
        python-version: [ '3.11' ]
    uses: UBC-MOAD/gha-workflows/.github/workflows/sphinx-linkcheck.yaml@main
    with:
      python-version: ${{ matrix.python-version }}
      conda-env-file: envs/environment-test.yaml
      conda-env-name: <test-env-name>
```


## `gha_workflows_checker.py` Script

Use the GitHub CLI tool to list GitHub Actions workflows in repositories with their
enabled/disabled status.

The Initial impetus for this script was to provide an easy way to check for GitHub's automatic
disabling of scheduled `sphinx-linkcheck` workflows in repos that haven't had activity for >60d.

Run in a terminal via:

```bash
pixi run check
```

Disabled workflows can be re-enabled with commands like:

```bash
pixi run gh -R UBC-MOAD/moad_tools workflow enable CodeQL
```
