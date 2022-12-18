# gha-workflows

[![Licensed under the Apache License, Version 2.0](https://img.shields.io/badge/license-Apache%202-cb2533.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Git on GitHub](https://img.shields.io/badge/version%20control-git-blue.svg?logo=github)](https://github.com/UBC-MOAD/gha-workflows)
[![Issue Tracker](https://img.shields.io/github/issues/UBC-MOAD/gha-workflows?logo=github)](https://github.com/UBC-MOAD/Reshapr/issues)


Reusable GitHub Actions workflows for MOAD repositories and workflow management tools

## Changes

### 2-Dec-2022

Changed to rely on the Slack github app workflows subscription feature to send workflow status 
notifications to Slack instead of the 8398a7/action-slack action.

Enable that feature with:

`/github subscribe org/repo workflows:{event:"pull_request","push" branch:"main"}`

ref: https://github.com/integrations/slack#actions-workflow-notifications


## How to Use the Workflows

YAML blobs to use the reusable workflows in other repositories.

**Note:** *Replace the `SHA` at the end of the `uses:` line with the SHA hash of the most recent commit
of the workflow.*

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
    uses: UBC-MOAD/gha-workflows/.github/workflows/auto-assign.yaml@SHA
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
        python-version: [ '3.10' ]
    uses: UBC-MOAD/gha-workflows/.github/workflows/sphinx-linkcheck.yaml@SHA
    with:
      python-version: ${{ matrix.python-version }}
      conda-env-file: envs/environment-test.yaml
      conda-env-name: moacean-parcels-test
```
