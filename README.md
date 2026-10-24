# meta: scripts for carrying fixes on top of upstream

This is an orphan branch: it shares no history with MBBSEmu's code and is never merged anywhere.
It's checked out as a worktree in `Meta/` of the main clone:

```bash
git worktree add Meta meta
echo Meta/ >> .git/info/exclude
```

## Branches

- `upstream/master`: mbbsemu/MBBSEmu, never committed to. Start every new branch from here.
- `master` on the fork: a mirror of `upstream/master`, fast-forwarded by `reintegrate --push`.
  There's deliberately no local `master` branch, so there's no stale copy to branch from.
- `fix/*`: one fix each, started from `upstream/master`. Most are or will be PRs upstream.
- `pr/<number>`: someone else's open PR, fetched from upstream's `refs/pull/<number>/head`.
- `seen`: what gets built and run. It's rebuilt from `upstream/master` plus every branch it
  carries, and never edited by hand. The name comes from git's own integration branch.

## Day to day

| To | Run |
|---|---|
| Pick up upstream and rebuild | `Meta/reintegrate` (add `--push` to back up `seen`, `--publish` for the Pi build) |
| Carry a new fix | `git switch -c fix/x upstream/master`, commit, then `git switch seen && git merge --no-ff fix/x` |
| Carry someone's PR | `git fetch upstream pull/<n>/head:pr/<n> && git switch seen && git merge --no-ff pr/<n>` |
| Check a new fix against the rest | `Meta/overlaps fix/x` |
| Stop carrying a fix | `git branch -D fix/x` (and `git push origin --delete fix/x` if it's pushed) |
| See what's carried | `git log --first-parent --merges --oneline upstream/master..seen` |

A branch whose PR was merged upstream drops out on the next rebuild by itself: its merge comes
out empty. If upstream merged only part of a fix, the rest drops out too, so read the
"dropped" lines and the test results.

## Conflicts

`reintegrate` builds on a temporary branch, `seen-new`, and only moves `seen` when every merge
succeeded. If a merge conflicts, it stops there. Resolve the conflict, `git commit`, and run it
again. rerere (turned on by the script) remembers the resolution and reapplies it on every
later rebuild.
