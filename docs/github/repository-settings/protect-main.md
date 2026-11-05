# Protect the main branch

**TL;DR:** add a branch ruleset for the default branch with **Restrict deletions**,
**Block force pushes**, **Require a pull request before merging** and
**Require status checks to pass before merging**. Leave the bypass list empty.

## Problem

Out of the box, anyone with write access to the repository can:

- push straight to `main`, skipping review and CI;
- `git push --force` and rewrite or drop history that others have already pulled;
- delete `main` by mistake.

On a repo where merging to `main` deploys (like this site), every one of these ships to production.

## Solution

Settings → Rules → **Rulesets** → **New ruleset** → **New branch ruleset**.

1. **Ruleset name:** `main`.
2. **Enforcement status:** **Active**.
3. **Target branches:** **Add target** → **Include default branch**.
4. **Bypass list:** leave empty (see [Bypass list](#bypass-list)).
5. Rules:

| Rule | What it stops |
|------|---------------|
| **Restrict deletions** | Deleting `main`. |
| **Block force pushes** | `git push --force` to `main`. |
| **Require a pull request before merging** | Any direct push. Every change has to arrive through a PR. |
| **Require status checks to pass before merging** | Merging a PR whose CI is red or still running. |
| **Require linear history** | Merge commits on `main`. Optional, see [Pull request settings](pull-requests.md#merge-methods). |

With **Require a pull request before merging** on, a direct push is rejected:

```text
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: - Changes must be made through a pull request.
```

### Pull request options

These sit under **Require a pull request before merging**. The right values depend on
whether anyone else can review your PRs.

=== "Solo"

    - **Required approvals:** `0`. GitHub never lets you approve your own PR,
      so `1` would block every merge.
    - Leave the other review options off.

    The PR still earns its keep: CI runs before the merge, and the diff gets one last look.

=== "Team"

    - **Required approvals:** `1` or more.
    - **Dismiss stale pull request approvals when new commits are pushed:** on.
      An approval covers the code that was reviewed, not what was pushed later.
    - **Require approval of the most recent reviewable push:** on.
      Stops the author from pushing a change after the last approval and merging it alone.
    - **Require review from Code Owners:** on, together with a `.github/CODEOWNERS` file.
    - **Require conversation resolution before merging:** on, so review comments are not lost.

    ```text
    # .github/CODEOWNERS: the last matching pattern wins
    *                    @my-org/maintainers
    /docs/security/      @my-org/security
    .github/workflows/   @my-org/platform
    ```

### Status checks

Under **Require status checks to pass before merging**, **Add checks** and pick the CI job by name.
For this repo it is `docs`, the job in `.github/workflows/deploy.yml` that runs `mkdocs build --strict`.
GitHub lists a check only after it has run at least once, so open a PR first.

**Require branches to be up to date before merging** makes the PR re-run CI on top of the latest `main`.

| Option | Pros | Cons |
|--------|------|------|
| Up to date: on | CI tested exactly what lands on `main`. | Every merge to `main` makes the other open PRs update and re-run CI. |
| Up to date: off | No extra rebases or CI runs. | Two PRs that pass alone can break `main` together. |

!!! tip "Recommendation"
    On for a solo or small repo: few PRs are open at once, so the cost is low.
    For a busy repo, look at a merge queue (rule **Require merge queue**) instead of making everyone rebase.

## Why it works

Rules are checked on GitHub's side, on every push and every merge, whatever client is used.
A local `pre-push` hook can be skipped with `--no-verify`. A ruleset cannot.

See [About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
and [Available rules for rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets).

## Trade-offs

### Rulesets or classic branch protection

GitHub has two features for this. Both are still supported and both can apply to the same branch.

| Option | Pros | Cons |
|--------|------|------|
| **Rulesets** (Settings → Rules → **Rulesets**) | Several rulesets can stack on one branch, the strictest rule wins. Can be switched off without deleting. Anyone with read access can see them. Export to JSON and import into another repo. Also cover tags. | Fewer guides online still show this UI. |
| **Branch protection rules** (Settings → **Branches**) | Older, so most tutorials use it. | One rule per branch. Admins are exempt unless you turn on **Do not allow bypassing the above settings**. Branches only. Only admins can see the rules. |

!!! tip "Recommendation"
    Rulesets for anything new. If a repo already has a branch protection rule, either convert it
    or keep it: GitHub enforces both, so you do not lose protection while you move.

### Bypass list

Whoever is on the bypass list skips every rule in the ruleset.

| Option | Pros | Cons |
|--------|------|------|
| Empty | The rules apply to everyone, you included. No "just this once" pushes to `main`. | An emergency fix also has to go through a PR. To get around it, an admin has to switch the ruleset to **Disabled**, which leaves a trace. |
| **Repository admin**, **Always allow** | Hotfix without waiting for CI. | Your own mistakes are no longer caught: it is too easy to push to `main` from the wrong terminal. |
| **Repository admin**, **For pull requests only** | Admin can merge a PR that is not approved or whose checks are red, but direct pushes are still blocked. | An unreviewed merge is still one click away. |
| A GitHub App or deploy key | A release bot can push version bumps or tags. | Whoever controls that token can push to `main` too. |

!!! tip "Recommendation"
    Empty. If a bot needs to push, add only that app, and only to the ruleset it needs.

### Signed commits

**Require signed commits** accepts only commits with a verified GPG, SSH or S/MIME signature.

| Option | Pros | Cons |
|--------|------|------|
| Off | Nothing to set up. | Anyone who has set `user.email` to your address can commit "as you". Write access is still needed to push. |
| On | Every commit on `main` is tied to a key on a GitHub account. | Everyone, and every bot, has to sign. Unsigned commits in a PR branch can block a squash merge even though GitHub signs the squash commit itself. Merging locally needs the result signed before the push. |

!!! tip "Recommendation"
    Off for a solo repo, on when several people or bots push and a forged author would matter.
    Sign your own commits either way, it costs one-time setup.

## Protect release tags too

A tag like `v1.2.0` can be deleted and pointed at another commit, and package managers
and GitHub Actions users follow tags. Add a second ruleset:
**New ruleset** → **New tag ruleset**, target `v*`, with **Restrict updates** and **Restrict deletions**.
Add **Restrict creations** if only a release workflow should create tags.

## Same rules on many repos

A ruleset can be exported to JSON from its page and imported into another repo
with **New ruleset** → **Import a ruleset**. The same JSON works with the API:

```bash
gh api repos/<owner>/<repo>/rulesets                         # list
gh api repos/<owner>/<repo>/rulesets --method POST --input main.json
```

```json
{
  "name": "main",
  "target": "branch",
  "enforcement": "active",
  "bypass_actors": [],
  "conditions": { "ref_name": { "include": ["~DEFAULT_BRANCH"], "exclude": [] } },
  "rules": [
    { "type": "deletion" },
    { "type": "non_fast_forward" },
    { "type": "pull_request", "parameters": {
        "required_approving_review_count": 0,
        "dismiss_stale_reviews_on_push": false,
        "require_code_owner_review": false,
        "require_last_push_approval": false,
        "required_review_thread_resolution": false } },
    { "type": "required_status_checks", "parameters": {
        "strict_required_status_checks_policy": true,
        "required_status_checks": [{ "context": "docs" }] } }
  ]
}
```

`deletion` is **Restrict deletions** and `non_fast_forward` is **Block force pushes**.
GitHub's [ruleset-recipes](https://github.com/github/ruleset-recipes) repo has more examples.

## Pitfalls

!!! warning "Private repos on GitHub Free"
    Rulesets and branch protection work on public repos on every plan,
    but on private repos only with GitHub Pro, Team or Enterprise.
    On a free private repo you can still create a ruleset, but GitHub saves it without enforcing it.

!!! warning "A required check that never runs"
    If the required check's workflow skips a PR (a `paths:` filter, a renamed job),
    the check stays "Expected" forever and the PR cannot be merged.
    Name the job, not the workflow, and keep the name stable.

!!! warning "Evaluate mode"
    **Evaluate** (dry-run: log what would be blocked) is GitHub Enterprise only.
    On other plans, test a new ruleset on a throwaway branch pattern before targeting `main`.
