# Privacy

What GitHub shows about you to anyone on the internet, and how to limit it.

**TL;DR:** commit with the GitHub no-reply email, hide the email on your profile,
and don't rewrite old public history just to remove an email: it is already copied elsewhere.

## Why bother

Public GitHub data is scraped all the time. A real email or name in a commit leads to:

- spam and recruiter mailing lists;
- targeted phishing ("your repo X has a security issue, log in here");
- linking your GitHub identity to your other accounts.

Whatever is in a public commit stays public forever: forks, clones,
[GH Archive](https://www.gharchive.org/) and [Software Heritage](https://www.softwareheritage.org/)
keep copies you cannot delete.

## What GitHub exposes

| Data | Where anyone can see it | How to limit it |
|------|-------------------------|-----------------|
| Commit email | `git log`, `<commit URL>.patch` | [Commit email](commit-email.md) |
| Commit name | `git log`, GitHub UI | [Commit email](commit-email.md#name) |
| Profile email, name, location, company | Your profile page | [Profile](profile.md) |
| Activity: when and how much you work | Contribution graph, activity feed | [Profile](profile.md#activity) |
| Time zone | Commit timestamps, profile local time | [Other leaks](other-leaks.md#time-zone-in-commits) |
| Public SSH and GPG keys | `github.com/<user>.keys`, `github.com/<user>.gpg` | [Other leaks](other-leaks.md#public-ssh-and-gpg-keys) |
| Email in published packages | PyPI, npm metadata | [Other leaks](other-leaks.md#package-metadata) |

## Checklist

- [ ] Settings → Emails: **Keep my email addresses private** is on.
- [ ] Settings → Emails: **Block command line pushes that expose my email** is on.
- [ ] `git config --global user.email` is your no-reply address.
- [ ] Settings → Public profile: **Public email** is "Don't show my email address".
- [ ] Your GPG key, if any, has no personal email in it.
- [ ] `pyproject.toml` / `package.json` of published packages have no personal email.
