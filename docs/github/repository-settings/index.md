# Repository settings

How to set up a GitHub repository so that `main` is protected, every change goes through a PR,
and the repo does not leak secrets or run untrusted code.

**TL;DR:** a ruleset on `main` that blocks deletion, force pushes and direct pushes, and requires CI;
squash merging only; every security feature under **Advanced Security** on;
a read-only `GITHUB_TOKEN` and actions pinned to a commit SHA.

The advice is for a solo or personal repo first. Where a team needs something different
(review approvals, CODEOWNERS), the page says so.

## Pages

| Page | Covers |
|------|--------|
| [Protect the main branch](protect-main.md) | Rulesets vs branch protection, blocking direct and force pushes, required PRs and CI, bypass list, tags. |
| [Pull request settings](pull-requests.md) | Merge methods, auto-merge, deleting merged branches. |
| [Security features](security.md) | Dependabot, secret scanning, push protection, code scanning, vulnerability reporting. |
| [GitHub Actions settings](actions.md) | `GITHUB_TOKEN` permissions, pinning actions, fork PRs, deploy secrets. |

## What your plan allows

| Feature | Public repo, any plan | Private repo, Free | Private repo, Pro / Team |
|---------|-----------------------|--------------------|--------------------------|
| Rulesets and branch protection | yes | no | yes |
| Dependabot alerts and updates | yes | yes | yes |
| Secret scanning, push protection, code scanning | yes, free | no | Team or Enterprise organization with GitHub Secret Protection / Code Security |
| Private vulnerability reporting | yes | no | no |

## Checklist

**Protect `main`** ([details](protect-main.md)):

- [ ] Branch ruleset `main` targets the default branch, **Enforcement status** is **Active**.
- [ ] **Restrict deletions** and **Block force pushes** are on.
- [ ] **Require a pull request before merging** is on. Required approvals: `0` solo, `1`+ team.
- [ ] **Require status checks to pass before merging** lists the CI job.
- [ ] Bypass list is empty, or holds only the bot that really needs to push.
- [ ] A tag ruleset protects `v*` tags from updates and deletion.

**Pull requests** ([details](pull-requests.md)):

- [ ] Only **Allow squash merging** is on, with **Pull request title and description**.
- [ ] **Automatically delete head branches** is on.
- [ ] **Always suggest updating pull request branches** and **Allow auto-merge** are on.

**Security** ([details](security.md)):

- [ ] **Dependabot alerts** and **Dependabot security updates** are on.
- [ ] `.github/dependabot.yml` updates at least `github-actions`.
- [ ] **Secret scanning** and **Push protection** are on.
- [ ] Code scanning: **CodeQL analysis** uses the **Default** setup.
- [ ] **Private vulnerability reporting** is on and `SECURITY.md` says how to report.

**Actions** ([details](actions.md)):

- [ ] **Workflow permissions** is **Read repository contents and packages permissions**.
- [ ] **Allow GitHub Actions to create and approve pull requests** is off.
- [ ] Third-party actions are pinned to a full commit SHA.
- [ ] Fork PR workflows need approval from a maintainer.
- [ ] Deploy secrets live in an environment, not in repository secrets.
