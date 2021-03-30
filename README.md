# Retype Build Action

A GitHub Action to build a [Retype](https://retype.com/) powered website. The output of this action is then made available to subsequent workflow steps, such as publishing to GitHub Pages using the [retypeapp/action-github-pages](https://github.com/retypeapp/action-github-pages) action.

## Introduction

This action runs `retype build` over the files in a repository to build a website in the form of a static html website that can be published to any website hosting solution available.

After the action completes, the action will export the `retype-output-root` value for the next steps to handle the output. The output files can then be pushed back to GitHub, or sent by FTP to another web server, or any other form of website publication target.

This action will look for a [`retype.json`](https://retype.com/configuration/project/) file in the repository root.

## Prerequisites

We highly recommend configuring the [actions/setup-dotnet](https://github.com/actions/setup-dotnet) step before `retypeapp/action-build`. This will install the tiny `dotnet` Retype package instead of the larger self-contained NPM package. 

Both the `dotnet` and `npm` packages run the exact same version of Retype. The size of the `dotnet` package is much smaller, so the action will setup faster.

## Usage

```yaml
steps:
- uses: actions/checkout@v2

- uses: actions/setup-dotnet@v1
  with:
    dotnet-version: 5.0.x

- uses: retypeapp/action-build
```

## Inputs

Configuration of the project should be done in the projects [`retype.json`](https://retype.com/configuration/project) file.

### `base`

The [`base`](https://retype.com/configuration/project#base) subfolder path appended to URL's.

The `base` is required if the target host will prefix a path to your website, such as the repository name with GitHub Pages hosting. 

For instance, https://example.com/docs/ would require `base: docs` to be configured. The path https://example.com/en/ would require `base: en` to be configured.

The `base` can also be configured in the project `retype.json` file.

### `license`

Specifies the license key to be used with Retype.

**WARNING**: Never save the `license` key value directly to your `retype.yaml` or `retype.json` files. Please use a GitHub Secret to store the value. For information on how to configure a secret, see [Encrypted Secrets](https://docs.github.com/en/actions/reference/encrypted-secrets).

## Examples

The following `retype.yaml` workflow file will serve as our starting template for most of the samples below.

```yaml
name: GitHub Action for Retype
on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  publish:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - uses: actions/setup-dotnet@v1
        with:
          dotnet-version: 5.0.x

      - uses: retypeapp/action-build@v1
```

Here are a few common workflow scenarios. 

### Most common setup

```yaml
steps:
  - uses: retypeapp/action-build
```

## Specify a custom `base` directory

If the output is not hosted from the website root folder, a `base` must be explicitly configured.

The `base` would typically be configured in the `retype.json` file.

```yaml
- name: Sets a variable with the repository name, stripping out owner/organization
  id: clean-repo-name
  shell: bash
  run: echo "::set-output name=repository_name::${GITHUB_REPOSITORY#${{ github.repository_owner }}/}"

- uses: retypeapp/action-build
  with:
    base: "${{ steps.clean-repo-name.outputs.repository_name }}"
```

The example above is useful to set up GitHub Pages using the `repo-owner.github.io/repo-name` path for hosting documentation built by Retype. For more information, see [Working with GitHub Pages](https://docs.github.com/en/github/working-with-github-pages).

## Specify a Retype license key

If a `license` key is required, please configure using a GitHub Secret.

```yaml
- uses: retypeapp/action-build
  with:
    license: ${{ secrets.RETYPE_LICENSE_KEY }}
```

For more information on how to set up and use secrets in GitHub actions, see [Encrypted secrets](https://docs.github.com/en/actions/reference/encrypted-secrets).

## Passing the output path to another action

It is possible to get the output path of this step to use in other steps or actions after the `action-build` is complete by using the `retype-output-root` value.

```yaml
- uses: retypeapp/action-build
  id: build1

- shell: bash
  env:
    RETYPE_BUILD_PATH: ${{ steps.build1.outputs.retype-output-root }}
  run: echo "Retype config is at '${RETYPE_BUILT_PATH}' and the actual output at '${RETYPE_BUILT_PATH}'."
```

Other Retype actions within the workflow may consume the output of this action by using the `RETYPE_OUTPUT_ROOT` environment variable.

It is required to upload the output with [actions/upload-artifact](https://github.com/actions/upload-artifact), as changes in the file system are not available across different GitHub action jobs. Then from the subsequent job(s), the artifact can be retrieved using the `download-artifact` action. 

The following sample demonstrates the [`upload-artifact`](https://github.com/actions/upload-artifact) and [`download-artifact`](https://github.com/actions/download-artifact) actions.

## Uploading the output as an artifact

To use the Retype output in another job within the same workflow, or let an external source download it, it is possible to use [`actions/upload-artifact`](https://github.com/actions/upload-artifact) to persist the files. The uploaded artifact can then be retrieved in another job or workflow using [`actions/download-artifact`](https://github.com/actions/download-artifact)

```yaml
- uses: retypeapp/action-build
  id: build1

- uses: actions/upload-artifact@v2
  with:
    path: ${{ steps.build1.outputs.retype-output-root }}
```

## Publishing to GitHub Pages

By using the Retype [retypeapp/action-github-pages](https://github.com/retypeapp/action-github-pages) action, the workflow can publish the output to a branch, or directory, or even a make a Pull Request. The website can then be hosted using [GitHub Pages](https://docs.github.com/en/github/working-with-github-pages/getting-started-with-github-pages).

The following sample demonstrates configuring the Retype `action-github-pages` action to publish to GitHub Pages:

```yaml
- uses: retypeapp/action-build

- uses: retypeapp/action-github-pages
  with:
    branch: retype
    update-branch: true
```
