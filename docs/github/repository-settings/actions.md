# GitHub Actions settings

**TL;DR:** make `GITHUB_TOKEN` read-only by default and grant write per job in the workflow,
don't let Actions approve PRs, pin third-party actions to a commit SHA,
and require approval before running workflows from fork PRs.

## Problem

A workflow runs code you did not write (every `uses:` line) with a token that can push to the repo.
The defaults on older repos make this worse:

- `GITHUB_TOKEN` gets read **and** write access to everything, in every job;
- `uses: some/action@v3` runs whatever commit the `v3` tag points to today.
  Whoever controls that action can move the tag;
- a workflow can approve a PR, which defeats the review rule in the [`main` ruleset](protect-main.md).

## Solution

Settings → **Actions** → **General**:

| Setting | Value |
|---------|-------|
| **Actions permissions** | **Allow OWNER, and select non-OWNER, actions and reusable workflows**, with **Allow actions created by GitHub** on and your list of allowed actions |
| **Require actions to be pinned to a full-length commit SHA** | on |
| **Approval for running fork pull request workflows from contributors** | **Require approval for first-time contributors** or stricter |
| **Workflow permissions** | **Read repository contents and packages permissions** |
| **Allow GitHub Actions to create and approve pull requests** | off |

Then give each job only what it needs in the workflow file:

```yaml
permissions: {}            # workflow default: no access at all

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@<full-length SHA>  # v4.2.2
      - run: make build

  deploy:
    if: github.event_name == 'push'
    needs: build
    runs-on: ubuntu-latest
    permissions:
      contents: write      # only this job can push
    steps:
      - ...
```

## Why it works

- The token setting is the default for every workflow. The `permissions:` key in YAML
  overrides it, so a job gets write access only where the file says so, and a reviewer sees it in the diff.
- A commit SHA cannot be moved. If the action's repo is taken over, you keep running the code you checked.
- Fork PRs are where untrusted code comes from. Approval means a maintainer looks at the PR before it runs on your runners.

See [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
and [Managing GitHub Actions settings for a repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository).

## Trade-offs

### Pin actions to a commit SHA

| Option | Pros | Cons |
|--------|------|------|
| Major tag (`@v4`) | Gets fixes automatically. Easy to read. | The tag can be moved to malicious code, and you run it on the next workflow run. |
| Exact tag (`@v4.2.2`) | Easy to read. | A tag is still just a pointer and can be moved. |
| Full commit SHA (`@11bd719...  # v4.2.2`) | Immutable: you run exactly the code you reviewed. | Unreadable without the comment. Never updates by itself, so you need Dependabot. |

!!! tip "Recommendation"
    SHA with the version as a comment, and the `github-actions` ecosystem in
    [Dependabot version updates](security.md#dependabot-version-updates): it bumps the SHA and the comment together.
    Actions from `actions/*` and `github/*` are lower risk, but the setting
    **Require actions to be pinned to a full-length commit SHA** applies to them too.

### Who can run workflows from forks

Public repos only. Private repos have a separate set of fork options, all off by default.

| Option | Pros | Cons |
|--------|------|------|
| **Require approval for first-time contributors who are new to GitHub** | Least friction for contributors. | Only stops brand-new accounts. |
| **Require approval for first-time contributors** | One look at each new contributor's first PR. | After one merged PR, a contributor's later PRs run without approval. |
| **Require approval for all external contributors** | Every fork PR is looked at before it runs. | Maintainer has to click **Approve and run** on every PR from a non-collaborator. |

!!! tip "Recommendation"
    **Require approval for first-time contributors** for a typical public repo.
    **Require approval for all external contributors** if workflows use self-hosted runners or anything expensive.

## Deploy secrets

- Put deploy secrets into an **environment** (Settings → **Environments**), not into repository secrets.
  An environment can require a reviewer and allow only certain branches, for example only `main`.
- For cloud deploys, use OpenID Connect (OIDC): the cloud trusts a short-lived token from GitHub,
  so there is no long-lived key to leak.

## Pitfalls

!!! warning "`pull_request_target`"
    It runs in the context of the base repo, with secrets and a write token, even for a fork PR.
    Never check out and run the PR's code in it. For a workflow that needs both
    untrusted code and secrets, split it into a `pull_request` workflow and a `workflow_run` one.

!!! warning "Switching the token to read-only breaks workflows"
    Workflows that push, comment, or create releases fail with `403` until their jobs
    declare the `permissions:` they need. Add the `permissions:` blocks first, then change the setting.

!!! warning "Workflow-level `permissions:` apply to every trigger"
    A `contents: write` at the top of a file that also runs on `pull_request`
    gives write access to runs for PRs from branches in the same repo too
    (fork PRs always get a read-only token). Put write permissions on the job that pushes,
    and keep that job behind `if: github.event_name == 'push'`.
