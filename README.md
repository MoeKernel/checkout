<p align="center">
  <a href="https://github.com/actions/checkout"><img alt="GitHub Actions status" src="https://github.com/actions/checkout/workflows/test-local/badge.svg"></a>
</p>

# Checkout V2

This action checks-out your repository under `$GITHUB_WORKSPACE`, so your workflow can access it.

Only a single commit is fetched by default, for the ref/SHA that triggered the workflow. Set `fetch-depth` to fetch more history. Refer [here](https://help.github.com/en/articles/events-that-trigger-workflows) to learn which commit `$GITHUB_SHA` points to for different events.

The auth token is persisted in the local git config. This enables your scripts to run authenticated git commands. The token is removed during post-job cleanup. Set `persist-credentials: false` to opt-out.

When Git 2.18 or higher is not in your PATH, falls back to the REST API to download the files.

# What's new

- Improved performance
  - Fetches only a single commit by default
- Script authenticated git commands
  - Auth token persisted in the local git config
- Creates a local branch
  - No longer detached HEAD when checking out a branch
- Improved layout
  - The input `path` is always relative to $GITHUB_WORKSPACE
  - Aligns better with container actions, where $GITHUB_WORKSPACE gets mapped in
- Fallback to REST API download
  - When Git 2.18 or higher is not in the PATH, the REST API will be used to download the files
  - When using a job container, the container's PATH is used
- Removed input `submodules`

Refer [here](https://github.com/actions/checkout/blob/v1/README.md) for previous versions.

# Usage

<!-- start usage -->
```yaml
- uses: actions/checkout@v2
  with:
    # Repository name with owner. For example, actions/checkout
    # Default: ${{ github.repository }}
    repository: ''

    # The branch, tag or SHA to checkout. When checking out the repository that
    # triggered a workflow, this defaults to the reference or SHA for that event.
    # Otherwise, defaults to `master`.
    ref: ''

    # Auth token used to fetch the repository. The token is stored in the local git
    # config, which enables your scripts to run authenticated git commands. The
    # post-job step removes the token from the git config. [Learn more about creating
    # and using encrypted secrets](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets)
    # Default: ${{ github.token }}
    token: ''

    # Whether to persist the token in the git config
    # Default: true
    persist-credentials: ''

    # Relative path under $GITHUB_WORKSPACE to place the repository
    path: ''

    # Whether to execute `git clean -ffdx && git reset --hard HEAD` before fetching
    # Default: true
    clean: ''

    # Number of commits to fetch. 0 indicates all history.
    # Default: 1
    fetch-depth: ''

    # Whether to download Git-LFS files
    # Default: false
    lfs: ''
```
<!-- end usage -->

# Scenarios

- [Checkout a different branch](#Checkout-a-different-branch)
- [Checkout HEAD^](#Checkout-HEAD)
- [Checkout multiple repos (side by side)](#Checkout-multiple-repos-side-by-side)
- [Checkout multiple repos (nested)](#Checkout-multiple-repos-nested)
- [Checkout multiple repos (private)](#Checkout-multiple-repos-private)
- [Checkout pull request HEAD commit instead of merge commit](#Checkout-pull-request-HEAD-commit-instead-of-merge-commit)
- [Checkout pull request on closed event](#Checkout-pull-request-on-closed-event)
- [Checkout submodules](#Checkout-submodules)
- [Fetch all tags](#Fetch-all-tags)
- [Fetch all branches](#Fetch-all-branches)
- [Fetch all history for all tags and branches](#Fetch-all-history-for-all-tags-and-branches)

## Checkout a different branch

```yaml
- uses: actions/checkout@v2
  with:
    ref: my-branch
```

## Checkout HEAD^

```yaml
- uses: actions/checkout@v2
  with:
    fetch-depth: 2
- run: git checkout HEAD^
```

## Checkout multiple repos (side by side)

```yaml
- name: Checkout
  uses: actions/checkout@v2
  with:
    path: main

- name: Checkout tools repo
  uses: actions/checkout@v2
  with:
    repository: my-org/my-tools
    path: my-tools
```

## Checkout multiple repos (nested)

```yaml
- name: Checkout
  uses: actions/checkout@v2

- name: Checkout tools repo
  uses: actions/checkout@v2
  with:
    repository: my-org/my-tools
    path: my-tools
```

## Checkout multiple repos (private)

```yaml
- name: Checkout
  uses: actions/checkout@v2
  with:
    path: main

- name: Checkout private tools
  uses: actions/checkout@v2
  with:
    repository: my-org/my-private-tools
    token: ${{ secrets.GitHub_PAT }} # `GitHub_PAT` is a secret that contains your PAT
    path: my-tools
```

> - `${{ github.token }}` is scoped to the current repository, so if you want to checkout a different repository that is private you will need to provide your own [PAT](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line).


## Checkout pull request HEAD commit instead of merge commit

```yaml
- uses: actions/checkout@v2
  with:
    ref: ${{ github.event.pull_request.head.sha }}
```

## Checkout pull request on closed event

```yaml
on:
  pull_request:
    branches: [master]
    types: [opened, synchronize, closed]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
```

## Checkout submodules

```yaml
- uses: actions/checkout@v2
- name: Checkout submodules
  shell: bash
  run: |
    # If your submodules are configured to use SSH instead of HTTPS please uncomment the following line
    # git config --global url."https://github.com/".insteadOf "git@github.com:"
    auth_header="$(git config --local --get http.https://github.com/.extraheader)"
    git submodule sync --recursive
    git -c "http.extraheader=$auth_header" -c protocol.version=2 submodule update --init --force --recursive --depth=1
```

## Fetch all tags

```yaml
- uses: actions/checkout@v2
- run: git fetch --depth=1 origin +refs/tags/*:refs/tags/*
```

## Fetch all branches

```yaml
- uses: actions/checkout@v2
- run: |
    git fetch --no-tags --prune --depth=1 origin +refs/heads/*:refs/remotes/origin/*
```

## Fetch all history for all tags and branches

```yaml
- uses: actions/checkout@v2
- run: |
    git fetch --prune --unshallow
```

## Lines of code added or modified vs. master
```yaml
# FAILS!!!
- run: git diff --diff-filter=am master HEAD > git_diff.txt || true
  # fatal: ambiguous argument 'master': unknown revision or path not in the working tree.
- run: git diff --diff-filter=am origin/master HEAD > git_diff.txt || true
  # fatal: ambiguous argument 'origin/master': unknown revision or path not in the working tree.
```

## File paths of files added or modified vs. master
```yaml
# FAILS!!!
- run: git diff --diff-filter=am --name-only master HEAD > git_diff.txt || true
  # fatal: ambiguous argument 'master': unknown revision or path not in the working tree.
- run: git diff --diff-filter=am --name-only origin/master HEAD > git_diff.txt || true
  # fatal: ambiguous argument 'origin/master': unknown revision or path not in the working tree.
```

# License

The scripts and documentation in this project are released under the [MIT License](LICENSE)
