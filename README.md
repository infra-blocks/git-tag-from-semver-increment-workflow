<<<<<<< HEAD
# git-tag-from-semver-increment-workflow
[![Git Tag Semver From Label](https://github.com/infrastructure-blocks/git-tag-from-semver-increment-workflow/actions/workflows/git-tag-semver-from-label.yml/badge.svg)](https://github.com/infrastructure-blocks/git-tag-from-semver-increment-workflow/actions/workflows/git-tag-semver-from-label.yml)
[![Update From Template](https://github.com/infrastructure-blocks/git-tag-from-semver-increment-workflow/actions/workflows/update-from-template.yml/badge.svg)](https://github.com/infrastructure-blocks/git-tag-from-semver-increment-workflow/actions/workflows/update-from-template.yml)
=======
# github-actions-workflow-template
[![Release](https://github.com/infrastructure-blocks/github-actions-workflow-template/actions/workflows/release.yml/badge.svg)](https://github.com/infrastructure-blocks/github-actions-workflow-template/actions/workflows/release.yml)
[![Trigger Update From Template](https://github.com/infrastructure-blocks/github-actions-workflow-template/actions/workflows/trigger-update-from-template.yml/badge.svg)](https://github.com/infrastructure-blocks/github-actions-workflow-template/actions/workflows/trigger-update-from-template.yml)
>>>>>>> template/master

This reusable workflow is meant to be used in conjunction with [check-has-semver-label-workflow](https://github.com/infrastructure-blocks/check-has-semver-label-workflow).
It leverages the [git-tag-semver-action](https://github.com/infrastructure-blocks/git-tag-semver-action) to manage a set of semantic versioning tags. The latter will update the
tag by creating a new version based on the semver increment provided and update the tags accordingly. Read the action
documentation for more information.

Which commit is being tagged depends on the event source. If the event has a `pull_request` payload, then the commit
used is `pull_request.head.sha`. Otherwise, it uses `github.sha`.

Details are reported as a [status report](https://github.com/infrastructure-blocks/status-report-action) at the end of the workflow.

## Inputs

|       Name       | Required | Description                                                                                                                                                                  |
|:----------------:|:--------:|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| semver-increment |   true   | The semver increment that indicates which release version will be produced. Should be one of "major", "minor" or "patch".                                                    |
|       skip       |  false   | A boolean indicating whether to skip the workflow. This is to workaround the required checks discrepancy when the workflow is skipped from the caller. It defaults to false. |

|     Name     | Required | Description                                                                                                                                                                                                                                                                      |
|:------------:|:--------:|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| github-token |  false   | The GitHub token utilized to push the tags. If pushing a tag matching a protection rule, this should be a PAT. Defaults to the $GITHUB_TOKEN otherwise. Note that the workflow still utilizes the $GITHUB_TOKEN for other operations, such as posting status report PR comments. |

## Outputs

| Name | Description                                                               |
|:----:|---------------------------------------------------------------------------|
| tags | A stringified JSON array containing the tags pushed against the git tree. |

## Permissions

|     Scope     | Level | Reason                                                       |
|:-------------:|:-----:|--------------------------------------------------------------|
|   contents    | write | Required to push tags.                                       |
| pull-requests | write | Required to post comments about the status of this workflow. |

## Concurrency controls

N/A

## Timeouts

N/A

## Usage

```yaml
name: Release

on:
  push:
    branches:
      - master

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  check-has-semver-label:
    permissions:
      pull-requests: write
    uses: infrastructure-blocks/check-has-semver-label-workflow/.github/workflows/workflow.yml@v2
  git-tag-from-semver-increment:
    uses: infrastructure-blocks/git-tag-semver-from-label-workflow/.github/workflows/workflow.yml@v1
    permissions:
      contents: write
      pull-requests: write
    with:
      semver-increment: ${{ needs.check-has-semver-label.outputs.matched-label }}
    secrets:
      github-token: ${{ secrets.PAT }} # Required to push against protected tags
```

### Releasing

The releasing is handled at git level with semantic versioning tags. Those are automatically generated and managed
by the [git-tag-semver-from-label-workflow](https://github.com/infrastructure-blocks/git-tag-semver-from-label-workflow).
