# Common github settings for Platform Projects

## Release drafter
This project contains the default configuration settings for [Release Drafter](https://github.com/marketplace/actions/release-drafter).

### PR labels

This is the mapping from pull request labels to release notes sections & semantic versioning:

| Section                | Label                               | Sem. Ver. | Notes           |
|------------------------|-------------------------------------|-----------|-----------------|
| *Breaking Changes*     | `breaking-change`                   | *major*   |                 |
| *New Features*         | `feature`                           | *minor*   |                 |
| *Improvements*         | `improvement`, `enhancement`        | *minor*   |                 |
| *Bug Fixes*            | `fix`, `bug`, `bugfix`              | *patch*   |                 |
| *Updated Dependencies* | `dependencies`                      | *patch*   | For @dependabot |
| *Documentation*        | `documentation`                     | *patch*   |                 |
| *Deprecation*          | `deprecation`                       | *minor*   |                 |
| *Maintenance*          | `maintenance`, `chore`, `internal`  | *patch*   |                 |

### Autolabeling

To help with the pr labeling, some rules have been configured based on branch and title conventions:

#### Branch convention
   `type/JIRA-ID_description`
* type: Type of development included in the branch. The possible values are
  * feature
  * feat
  * improvement
  * fix
  * bugfix
  * bug
  * hotfix
  * dependabot
  * docs
  * deprecation
  * terraform
  * helm
* JIRA-ID: Jira ID of the task being resolved.
* description: Describes the branch content.
  
ex: `feature/JIRA-1234_new_super_cool_tool`

#### PR title convention
Following conventional commits (https://www.conventionalcommits.org/en/v1.0.0/) standard also for PR titles
`type(scope)!: Description`
* type: (See Branch convention type)
* scope: (Optional) Specify a submodule where the changes are being applied.
* !: Exclamation symbol specify that the change is also a breaking change.
* description: Describes the branch content

ex:
* `feature: JIRA-1234 New super cool feature` Just a new feature
* `improvement!: JIRA-1234 New super cool improvement` Improvement that breaks with the previous version (!)
* `improvement(IAM chart): JIRA-1234 Improvement on the IAM chart submodule`

#### Autolabels configured

| Label             | Convention                                   |
|-------------------|----------------------------------------------|
| *breaking-change* | ! after the type                             |
| *feature*         | type `feature` or `feat`                     |
| *improvement*     | type `improvement`                           |
| *bug*             | type `bug`, `bugfix`, `fix`, `hotfix`        |
| *dependencies*    | type `dependabot`                            |
| *docs*            | if any .md file is change or, if type `docs` |
| *deprecation*     | type `deprection`                            |
| *terraform*       | type `Terraform`                           |
| *helm*            | type `Helm`                                |

As a remider this is the semantic version definition:

|Release|Format|
|-------|------|
|*Major*|**X**.y.z|
|*Minor*|x.**Y**.z|
|*Patch*|x.y.**Z**|

### Add Release drafter workflow on your project

#### Standalone
Add a new github workflow for the release drafter `.github/workflows/release-drafter.yml`:
```yaml
name: Release Drafter

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  update_release_draft:
    runs-on: ubuntu-latest
    steps:
      - uses: release-drafter/release-drafter@v5
        with:
          # (Optional) specify config name to use, relative to .github/. Default: release-drafter.yml
          config-name: release-drafter.yml
        env:
          GITHUB_TOKEN: ${{ secrets.ORGANIZATION_TOKEN }}
```

Note that the token needs organization wide access, because it needs access to this repo

#### Include as part of your CI/CD github workflow
> As each CI/CD could have its own particularities, make sure how each step can be adapted for your pipeline

Complete your trigger events with [push] and [pull_request] in case there aren't already there
```yaml
on:
  push:
    branches:
      - main
  pull_request:
```

Add a new job in your `jobs` list to perform the release-drafter
```yaml
update_release_draft:
    runs-on: ubuntu-latest
    steps:
      - uses: release-drafter/release-drafter@v5
        with:
          # (Optional) specify config name to use, relative to .github/. Default: release-drafter.yml
          config-name: release-drafter.yml
        env:
          GITHUB_TOKEN: ${{ secrets.ORGANIZATION_TOKEN }}
```

This job should be only executed when you need to draft the release notes, which is in most cases, on push to main or on pull_request, so add an if condition to ensure it.
```yaml
if: github.ref_type == 'branch'
```
with this condition you will skip `tag` events.

Finally add a last condition to the first job of your pipeline so yo skip pull_request executions (as you have probably added just release drafter).
```yaml
if: github.event_name != 'pull_request'
```

### Add Release Drafter configuration file
Add release drafter configuration (`.github/release-drafter.yml`) pointing to this repo and overwrite the template title:
```
_extends: .github
name-template: 'Module Name $RESOLVED_VERSION'
```
