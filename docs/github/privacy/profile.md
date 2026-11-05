# Profile and activity

**TL;DR:** hide the public email, fill in only the profile fields you want strangers to see,
and decide on purpose whether your activity is visible.

## Profile fields

Settings → Public profile. Every field here is public and indexed by search engines.

| Field | Recommendation |
|-------|----------------|
| Public email | "Don't show my email address". People who need to reach you can open an issue. |
| Name | Real name if GitHub is part of your professional profile, otherwise a handle. |
| Location | Country or city at most. |
| Company, social links | Only if you want them connected to this account. |
| Local time | Shows your time zone. Leave it off unless you want people to know when you are online. |

## Activity

### Private contributions

Profile → **Contribution settings** above the contribution graph → **Private contributions**.

The graph then also counts commits to private repos. Repo names stay hidden.

- **Pros:** the graph reflects your real work, useful when most of it is in private repos.
- **Cons:** shows when and how much you work: weekends, holidays, gaps.

### Private profile

Settings → Public profile → Contributions & Activity → **Make profile private and hide activity**.

Hides the contribution graph, activity feed, achievements, follower counts,
organizations, stars and the sponsor button. Keeps visible the README, bio and avatar.

- **Pros:** strangers cannot track your activity from one page.
- **Cons:** you cannot receive GitHub Sponsors. Your commits, issues and comments in public repos
  stay visible there, and some data is still reachable through the API.
  It does not hide what happened before you turned it on.

!!! note
    A private profile hides the overview, not the work. Anyone can still find
    your public commits and comments repo by repo.
