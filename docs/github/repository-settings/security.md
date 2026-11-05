# Security features

**TL;DR:** on a public repo, turn on everything under Settings → **Advanced Security**:
Dependabot alerts and security updates, secret scanning with push protection, CodeQL default setup,
and private vulnerability reporting. It is free for public repos.

## Problem

A repo is attacked through what it already contains:

- a dependency with a known CVE that nobody noticed;
- an API key committed "just for a test";
- a vulnerability that a stranger finds, and then reports in a public issue
  because there is no private way to report it.

## Solution

Settings → **Advanced Security**:

| Setting | What it does |
|---------|--------------|
| **Dependabot alerts** | Alerts when a dependency in your lockfile or manifest has a known vulnerability. |
| **Dependabot security updates** | Opens a PR that bumps the vulnerable dependency to the fixed version. |
| **Dependabot version updates** | Opens PRs for all new versions, on a schedule. Configured in `.github/dependabot.yml`, see below. |
| **Secret scanning** | Finds committed tokens and keys from known providers and alerts you. Many providers revoke a leaked token automatically. |
| **Push protection** | Rejects a push that contains a detected secret, before it is in history. |
| **CodeQL analysis** → **Set up** → **Default** | Code scanning: static analysis on every PR and push, results in the Security tab. |
| **Private vulnerability reporting** | Adds a **Report a vulnerability** button to the Security tab, so a reporter can open a private advisory. |

Then add a `SECURITY.md` in the repo root or in `.github/`. It shows up in the Security tab:

```markdown
# Security policy

Please report vulnerabilities privately with
**Security → Report a vulnerability** on this repository, not in a public issue.
We reply within 7 days.
```

### Dependabot version updates

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
  - package-ecosystem: "uv"   # or pip, npm, docker...
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      all:
        patterns: ["*"]     # one PR for all updates instead of one per package
```

The `github-actions` entry also keeps actions pinned to a commit SHA up to date,
see [Pin actions to a commit SHA](actions.md#pin-actions-to-a-commit-sha).

## Why it works

Each feature catches a problem at a different point:

- push protection: before the secret is in history;
- code scanning and the required status checks: before the PR is merged;
- Dependabot: after the merge, when a new CVE is published for code you already ship;
- private reporting: when someone else finds what all of the above missed.

See [Quickstart for securing your repository](https://docs.github.com/en/code-security/getting-started/quickstart-for-securing-your-repository)
and [About push protection](https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection).

## Trade-offs

### Dependabot version updates

Security updates fix known CVEs only. Version updates bump everything, which is a different trade:

| Option | Pros | Cons |
|--------|------|------|
| Security updates only | Few PRs, each fixes a real problem. | Everything else falls behind. When a CVE fix lands in a major version, you have to jump several majors at once. |
| Version updates, weekly, grouped | Small regular steps. Old dependencies don't pile up. | A PR every week to review and merge. Needs CI you trust. |
| Version updates, one PR per package | Easy to find which bump broke the build. | Lots of PRs. Most people stop reading them. |

!!! tip "Recommendation"
    Security updates on everywhere. Version updates weekly and grouped for repos that have CI,
    and always for `github-actions`, since actions run with access to your secrets.

## Pitfalls

!!! warning "Private repos"
    Secret scanning, push protection and code scanning are free on public repos only.
    On private repos they need an organization on GitHub Team or Enterprise with
    GitHub Secret Protection or GitHub Code Security. Dependabot works on every plan.

!!! warning "Push protection can be bypassed"
    Anyone with write access can push anyway by picking a reason:
    **It's used in tests**, **It's a false positive** or **I'll fix it later**.
    A secret that has been pushed is leaked: revoke it, don't just delete the commit.

!!! note "Push protection for users"
    Separate from the repo setting, GitHub blocks pushes of secrets *by you* to any public repo.
    It is on by default in your personal settings. Keep it on.
