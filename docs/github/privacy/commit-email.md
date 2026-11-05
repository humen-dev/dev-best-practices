# Hide your email in commits

**TL;DR:** set `user.email` to your GitHub no-reply address and turn on both email
privacy settings. Leave old public history as it is unless nobody has cloned the repo yet.

## Problem

Git writes the author email into every commit. On GitHub anyone can read it,
even when the web UI shows only your username:

```bash
git clone https://github.com/<owner>/<repo>.git && git log --format='%an <%ae>'
curl -s https://github.com/<owner>/<repo>/commit/<sha>.patch | head -3
```

This also applies to private repositories: everyone with access sees the email,
and it becomes public the day the repo is made public.

## Solution

### 1. Find your no-reply address

Settings → Emails, under **Keep my email addresses private**. Or:

```bash
gh api user --jq '"\(.id)+\(.login)@users.noreply.github.com"'
```

Accounts created after July 18, 2017 get the `ID+USERNAME@users.noreply.github.com` form.
Older accounts may have `USERNAME@users.noreply.github.com`.

!!! warning "Use the ID-based form"
    Commits with the old `USERNAME@` form stop linking to your account if you rename it.
    The ID-based form survives a rename. To get it, turn **Keep my email addresses private**
    off and on again.

### 2. Turn on both settings

Settings → Emails:

- **Keep my email addresses private**: commits made in the web UI
  (merging a PR, editing a file) use the no-reply address.
- **Block command line pushes that expose my email**: GitHub rejects a push
  if a commit contains your private email, with `GH007: Your push would publish a private email address`.
  This catches a misconfigured machine before the email goes public.

### 3. Set the email in Git

```bash
git config --global user.email "12345678+username@users.noreply.github.com"
```

Check the last commit:

```bash
git log -1 --format='%an <%ae>'
```

## Which email to use

| Option | Pros | Cons |
|--------|------|------|
| GitHub no-reply | Nothing to harvest. Commits still count on your profile. | Nobody can email you from `git log`. Links only to GitHub, not GitLab or others. |
| Public alias (`dev@yourdomain.com`) | People can contact you. You can filter or drop it later. | Still harvested, so expect spam. You need a domain or alias service. |
| Plus address (`me+github@gmail.com`) | Easy to filter. | Bots strip `+github` and get your real address. |
| Real personal email | No setup. | Spam and phishing, permanently. |

!!! tip "Recommendation"
    No-reply for GitHub. Use a real address only where the workflow needs it,
    for example patches sent by email to a mailing list.

## Work and personal identities

One global email is wrong as soon as you have work repos. Two ways to switch it:

=== "By folder"

    ```ini
    # ~/.gitconfig
    [user]
        name = Your Name
        email = 12345678+username@users.noreply.github.com

    # /i = case-insensitive, needed on Windows
    [includeIf "gitdir/i:~/work/"]
        path = ~/.gitconfig-work
    ```

    ```ini
    # ~/.gitconfig-work
    [user]
        email = you@company.com
    ```

    Pros: simple. Cons: a work repo cloned outside `~/work/` gets the personal email.

=== "By remote URL"

    ```ini
    # ~/.gitconfig, needs Git 2.36+
    [includeIf "hasconfig:remote.*.url:git@github.com:my-company/**"]
        path = ~/.gitconfig-work
    ```

    Pros: works wherever the repo lives. Cons: does not apply before `origin` is added,
    so the very first commit of a new repo can get the wrong email.

=== "No default at all"

    ```bash
    git config --global --unset user.email
    git config --global user.useConfigOnly true
    ```

    Git refuses to commit until you set `user.email` in the repo.
    Pros: never the wrong identity. Cons: one extra command in every new repo.

## Name

`user.name` is just as public. A real name helps employers and colleagues find your work.
A handle keeps your GitHub separate from your real identity. Pick one on purpose.

## Old commits

Changing the settings affects only new commits. Old ones keep the old email.

### Leave them

- **Pros:** nothing breaks.
- **Cons:** the email stays in history.

For a public repo this is usually the right call: forks, clones and archives
already have the old commits, so a rewrite does not un-publish the email.

### Rewrite them

```text
# mailmap.txt: new identity, then old email
Your Name <12345678+username@users.noreply.github.com> <old@example.com>
```

```bash
git filter-repo --mailmap mailmap.txt
git remote add origin git@github.com:<owner>/<repo>.git  # filter-repo removes origin on purpose
git push --force --all && git push --force --tags
```

- **Pros:** the email is gone from the repo's current branches.
- **Cons:** every commit SHA changes. Clones, forks and open PRs break.
  Commit signatures are lost. Old commits stay reachable by SHA on GitHub
  until GitHub Support purges them, and forks keep them anyway.

!!! tip "When a rewrite is worth it"
    A private repo you are about to make public, or a repo nobody else has cloned.

!!! note "`.mailmap` is not a fix"
    A `.mailmap` file in the repo only changes what `git log` and `git shortlog` print.
    The commits still contain the old email.
