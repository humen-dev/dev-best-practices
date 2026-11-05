# Other leaks

Places that expose personal data even after you fixed your commit email and profile.

## Public SSH and GPG keys

Anyone can download the public keys of any GitHub user:

```bash
curl https://github.com/<user>.keys
curl https://github.com/<user>.gpg
```

**GPG:** a GPG key contains its user IDs, so `<user>.gpg` shows every email in the key.
If you sign commits with GPG and use the no-reply address, create the key with
the no-reply address only.

**SSH:** a public key is safe to share, but the same key on many servers links your accounts
together. A server can check which of your GitHub keys you offer when you connect.

| Option | Pros | Cons |
|--------|------|------|
| One SSH key everywhere | Simple. | Your GitHub account can be linked to every other server you use. |
| One key per service | Nothing to link. | More keys to manage and rotate. |

With separate keys, stop SSH from offering every key to every server:

```text
# ~/.ssh/config
Host github.com
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes
```

!!! tip
    Signing commits with an SSH key instead of GPG avoids putting an email into a key at all.

## Time zone in commits

Every commit stores the UTC offset of your machine, for example `+0200`.
Together with the commit times this shows your region and working hours.

Committing in UTC hides the offset:

```bash
TZ=UTC git commit
```

- **Pros:** the offset no longer points to your region.
- **Cons:** local `git log` shows UTC. Commit times still show *when* you work.

Most people accept the offset. Bother only if your location really must stay secret.

## Package metadata

Publishing a package copies its metadata to PyPI or npm, where it stays even after
you delete the package:

```toml
# pyproject.toml
authors = [{ name = "Your Name", email = "you@personal.com" }]  # public forever
```

Use a project address or the no-reply address, or leave `email` out.

## Secret gists are not private

A secret gist is only unlisted. Anyone with the URL can open it,
and the URL leaks through browser history, chat previews and logs.
Use a private repository for anything sensitive.

## Logs and screenshots in issues

Stack traces and terminal output often contain your OS username, machine name,
internal hostnames or tokens:

```text
File "C:\Users\john.smith\work\acme-billing\app.py", line 42
```

Read pasted output before posting, and crop screenshots.
