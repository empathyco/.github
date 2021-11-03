# Common github settings for data projects

## Release drafter
This project contains the default configuration settings for [Release Drafter](https://github.com/marketplace/actions/release-drafter).

#### PR labels
This is the mapping from pull request labels to release notes sections & semantic versioning:

|Section|Label|Sem. Ver.| Notes|
|-------|-----|---------|------|
|*Breaking Changes*|`breaking-changes`| *major*
|*New Features*|`feature`, `enhancement`| *minor*
|*Bug Fixes*|`bugfix`, `bug`| *patch*
|*Maintenance*|`maintenance`| *patch*
|*Updated Dependencies *|`dependencies`|*patch*| For @dependabot
|*Documentation*|`documentation`| *patch*

As a remider this is the semantic version definition:

|Release|Format|
|-------|------|
|*Major*|**X**.y.z|
|*Minor*|x.**Y**.z|
|*Patch*|x.y.**Z**|

#### Activate action
Add release drafter workflow `.github/workflows/release-drafter.yml`:
```
name: Release Drafter

on:
  push:
    # branches to consider in the event; optional, defaults to all
    branches:
      - main

jobs:
  update_release_draft:
    runs-on: ubuntu-latest
    steps:
      # Drafts your next Release notes as Pull Requests are merged into "master"
      - uses: release-drafter/release-drafter@v5
        with:
          # (Optional) specify config name to use, relative to .github/. Default: release-drafter.yml
          config-name: release-drafter.yml
        env:
          GITHUB_TOKEN: ${{ secrets.ORGANIZATION_TOKEN }}
```

Note that the token needs organization wide access, because it needs access to this this repo

#### Configure Action
Add release drafter configuration (`.github/release-drafter.yml`) pointing to this repo and overwrite the template title:
```
_extends: platform-github-settings
name-template: 'Module Name $RESOLVED_VERSION'
```
