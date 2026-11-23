# Make a private repository public

**TL;DR:** before you flip the switch, scan the whole history for secrets, decide about
the author email in old commits, and review everything else that becomes public with the code.
Flip visibility last, then turn on the security features and re-check your rulesets.

## Problem

**Change visibility** publishes everything at once, and you cannot take it back:

- every commit on every branch, with author names and emails;
- every file that was ever committed, even if a later commit deleted it;
- issues, pull requests, wiki, releases and their assets;
- GitHub Actions history and logs: "Actions history and logs will be visible to everyone";
- copied third-party content, now redistributed in public.

Forks, clones and archives copy a public repo within minutes.
A secret or an email found after the switch is already out.

## Solution

Work through the steps in order. Steps 1–3 are cheap while the repo is still private
and expensive or impossible after.

### 1. Scan the whole history for secrets

GitHub secret scanning is free only on public repos, so on a private repo you have to scan yourself,
before the switch. Scan all branches and tags, not just the current tree:

```bash
gitleaks git --log-opts="--all" -v .
```

!!! warning "A found secret is a leaked secret"
    If the scan finds a real token or key, revoke and rotate it first.
    Removing it from history is optional after that; a revoked secret is harmless.

### 2. Decide about the email in old commits

```bash
git log --all --format='%an <%ae>' | sort | uniq -c
```

If a personal email shows up, this is the last moment when a history rewrite is cheap:
nobody outside has cloned the repo yet. After the switch, a rewrite no longer
un-publishes anything. See [Hide your email in commits](../privacy/commit-email.md#old-commits)
for both options and the rewrite commands.

Also set up the account before the first public push:
[Commit email](../privacy/commit-email.md) (**Keep my email addresses private**,
**Block command line pushes that expose my email**, `user.email` = no-reply address).

### 3. Review what else becomes public

- [ ] **Actions logs.** Open old workflow runs. Delete runs whose logs printed tokens,
      internal URLs or customer data (**Actions** → run → **⋯** → **Delete workflow run**).
- [ ] **Issues, pull requests, wiki, releases.** Nothing internal or personal in them.
- [ ] **Third-party content.** Copied files have a licence that allows redistribution,
      and the repo credits them (for example a `THIRD_PARTY_NOTICES.md`).
- [ ] **Docs that assume a private repo.** README install steps that need authentication
      (`gh auth setup-git`, a token), "this repository is private" in `SECURITY.md`.
- [ ] **Config and fixtures.** No real hostnames, IPs or internal service names in example files.

### 4. Change visibility

Settings → General → **Danger Zone** → **Change visibility**, choose public, type the repository
name, click **I have read and understand these effects**, then **Make this repository public**.

### 5. Right after the switch

- [ ] Turn on everything in [Security features](security.md). Secret scanning now also scans
      the full history: check **Security and quality** → **Secret scanning** for alerts.
- [ ] Re-check [Protect the main branch](protect-main.md): push rulesets are disabled by the switch.
- [ ] If the repo publishes a site: Settings → **Pages** → **Source** → **GitHub Actions**
      (or the branch you deploy from).

## Why it works

The order follows what can still be undone. While the repo is private, a secret can be rotated
and history rewritten before anyone copies it. After the switch the same mistake is permanent,
and the only fix is to revoke what leaked. Security features come last because on a free plan
most of them only become available once the repo is public.

See [Setting repository visibility](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility)
for the full list of effects.

## Trade-offs

### Public repo vs. a private repo on a paid plan

The usual reason to go public is a free feature: GitHub Pages, secret scanning, code scanning, rulesets.

| Option | Pros | Cons |
|--------|------|------|
| Make it public | Pages, secret scanning, push protection, CodeQL and rulesets are free. Others can read, fork and report issues. | Code, history and Actions logs are public for good. |
| Stay private on GitHub Pro | Code stays private. Pages and rulesets work. | Paid. A Pages site is still public: only Enterprise Cloud can restrict who sees it. Secret scanning and code scanning are not included. |
| Stay private, publish the site elsewhere | Code stays private. | One more service to run, and the site content is public anyway. |

!!! tip "Recommendation"
    If the content would end up public anyway (a docs site, a plugin catalog that shows every file),
    make the repo public and follow the steps above. Keep it private only when the code itself
    must stay secret.

## Pitfalls

!!! warning "Stars and watchers"
    Changing visibility in either direction erases stars and watchers.
    Switching back to private also leaves existing public forks public, detached from your repo.

!!! warning "A regex search is not a secret scan"
    `git log -p | grep -i token` misses encoded and multi-line secrets. Use a scanner with
    provider patterns (gitleaks, trufflehog), then let GitHub secret scanning check again after the switch.

!!! note "Workflows from other repositories"
    If workflows call reusable workflows from another repository in your organization,
    the public logs show that repository's name and file paths.
