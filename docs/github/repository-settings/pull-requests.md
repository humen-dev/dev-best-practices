# Pull request settings

**TL;DR:** allow squash merging only, with **Pull request title and description** as the commit message.
Turn on **Automatically delete head branches** and **Always suggest updating pull request branches**.

## Problem

By default a repo allows all three merge methods. Each person picks a different one,
so `main` ends up with a mix of merge commits, squashed commits and rebased WIP commits like `fix typo`.
Merged branches pile up in the branch list.

## Solution

Settings → **General** → **Pull Requests**:

| Setting | Value |
|---------|-------|
| **Allow merge commits** | off |
| **Allow squash merging** | on, default message **Pull request title and description** |
| **Allow rebase merging** | off |
| **Always suggest updating pull request branches** | on |
| **Allow auto-merge** | on |
| **Automatically delete head branches** | on |

## Why it works

- One merge method means one shape of history, and nobody has to choose.
- **Pull request title and description** puts the reviewed summary into the commit,
  instead of a list of commit messages nobody reviewed.
- **Always suggest updating pull request branches** shows an **Update branch** button
  whenever the PR is behind `main`, so you do not have to merge `main` in by hand.
- **Allow auto-merge** adds **Enable auto-merge** to a PR: it merges itself once the required
  reviews and checks from the [ruleset](protect-main.md) pass. It has no effect without such rules.
- **Automatically delete head branches** deletes the PR branch after merging.
  A deleted branch can be restored from the closed PR.

See [About merge methods on GitHub](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github).

## Trade-offs

### Merge methods

| Option | Pros | Cons |
|--------|------|------|
| Squash | One commit per PR on `main`: easy to read, revert and bisect. WIP commits in the branch don't matter. | The individual branch commits are not on `main`. Branches stacked on the PR branch need a rebase after the merge. |
| Merge commit | Keeps every commit and shows where the branch joined. Works with stacked branches. | `main` gets every `fix typo` commit plus a merge commit per PR. `git log` is hard to read without `--first-parent`. |
| Rebase | Linear history and every commit kept. | Only good if every commit in the PR is clean. Commits are rewritten, so their SHAs change and the author's signatures are lost. |

!!! tip "Recommendation"
    Squash only for most repos. A PR is the unit of review, so it is also the unit on `main`.
    Allow merge commits too only if you keep long-lived branches (for example `release/*`)
    that must be merged back without being flattened.

If you allow squash or rebase only, you can also turn on **Require linear history** in the
[`main` ruleset](protect-main.md#solution), so a merge commit cannot get in from the command line.

## Pitfalls

!!! warning "Squash and long-lived branches"
    After a squash merge, the branch's own commits are not ancestors of `main`.
    Keep working on the same branch and the next PR shows the old changes again.
    Start a new branch from `main` after each merge.

!!! warning "Auto-delete and forks"
    For a PR from a fork, GitHub checks the fork's setting, not yours.
    Branches with a ruleset that has **Restrict deletions** are not deleted either.
