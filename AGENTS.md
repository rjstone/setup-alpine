# AGENTS.md - rjstone/setup-alpine

Instructions for AI agents (and a reminder for the owner) working in this fork of
[jirutka/setup-alpine](https://github.com/jirutka/setup-alpine). This file lives on the
`rjstone` branch, which is where all work in this fork happens.

## Why this fork exists

`rjstone/alpine-make-vm-image` builds its DigitalOcean Droplet image with this action and
needed fixes faster than upstream ships them:

| Change | Commit | Why |
| --- | --- | --- |
| node24 chroot teardown (`gacts/run-and-post-run`) | `5d001b9` (#1) | `webiny/action-post-run@3.1.0` is node20 and caused a Node 20 deprecation warning on every run. |
| `sudo env` instead of `sudo -E` | `12906b4` (#3) | Ubuntu 26.04 runners (`ubuntu-latest` since Nov 2026) ship sudo-rs, which ignores a bare `-E`; every `INPUT_*` variable was dropped. |

Both are candidates to offer upstream (see *Upstream pull requests*).

## Branches

| Branch | Role | Rules |
| --- | --- | --- |
| `master` | Mirror of upstream `jirutka/setup-alpine` `master`. No commits of our own. | Keep in sync with upstream only (GitHub "Sync fork", or fast-forward). |
| `rjstone` | **Working branch.** All of our changes land here first. | No restrictions. Agents commit and push here. |
| `upstream/<topic>` (suggested name) | Short-lived branch cut from `master` to carry one change upstream. | Cherry-pick from `rjstone`; open the PR against `jirutka/setup-alpine`. |
| `v1` | Personal release line, consumed by our other repos. Not offered upstream. | Changes only through pull requests. |

The owner adjusts branch protection rules over time. Don't treat a rule described here as
the current one: if a push is refused, report it rather than looking for a way around it.

### Flow of a change

1. Commit to `rjstone`. CI runs on every push (any branch), across `ubuntu-latest`
   (26.04, sudo-rs), `ubuntu-24.04` and `ubuntu-22.04`, three Alpine branches and two arches.
2. **Into `v1`:** open a PR from `rjstone` (or a topic branch off it) into `v1`. After the
   merge, the owner tags a release on `v1`.
3. **Upstream:** cut `upstream/<topic>` from `master`, `git cherry-pick` the commit(s) from
   `rjstone`, push, and open a PR to `jirutka/setup-alpine` `master`. Keep each upstream
   PR to one change, and drop fork-only details (pinned commit hashes in comments,
   references to our repos).
4. When upstream merges something, sync `master`, then merge `master` into `rjstone`
   (and through a PR into `v1`) so the fork does not carry a duplicate of the change.

## Releases and tags

- Tags are cut on `v1` and use a **lowercase** `v`: `v1.4.2`, `v1.4.3`. (A `V1.4.3` was
  created once by a phone's autocapitalisation and replaced; check the case.)
- Upstream's tags (`v1.0.0` ... `v1.4.1`) are kept as they are. Our releases continue the
  numbering: `v1.4.2` = upstream `v1.4.1` + node24 teardown; `v1.4.3` = `v1.4.2` + `sudo env`.
- Consumers pin by **commit hash** with the tag in a comment, so a moved tag cannot change
  a build:
  ```yaml
  uses: rjstone/setup-alpine@23b837e7f47a9c37f29276612a7bb0aee5d8c7d4 # v1.4.3
  ```
  After a release, update the pin in `rjstone/alpine-make-vm-image`
  (`.github/workflows/ci.yml`; branches `do-droplet-image` and
  `do-droplet-agent-gateway-image`).

## Current state and open work

As of 2026-11-12:

- **`master` is not yet a clean mirror.** It is upstream `ae3b3dd` plus our `5d001b9` (#1)
  and `12906b4` (#3), from before this policy. Both are already on `rjstone` and `v1`.
  Making `master` match upstream means resetting it to upstream and force-pushing, which
  is the owner's decision; agents must not do it.
- Neither change has been offered upstream yet.
- `v1` = `23b837e` = tag `v1.4.3`.

## Rules for agents

- Work on `rjstone`. Don't push to `master` or `v1`, and don't create or move tags; the
  owner does releases.
- Never force-push, rebase or amend anything already pushed, on any branch, unless the owner
  asks for that specific operation.
- Don't open PRs against `jirutka/setup-alpine` without the owner's go-ahead: they are
  public and speak for the owner.
- When the action's shell code changes, keep it working under both classic sudo and sudo-rs;
  don't rely on `sudo -E` or on `--preserve-env` without a list.
- Keep this file current when the policy or the branch state changes.
