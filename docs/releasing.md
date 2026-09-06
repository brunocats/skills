# Releasing

Releases are automated. The maintainer merges pull requests; version numbers,
changelogs, tags and GitHub releases happen without intervention.

## How it works

[release-please](https://github.com/googleapis/release-please) reads the commit
history on `main` and keeps a standing **release pull request** up to date. That
pull request holds the next version number and the changelog entries generated
from the commits since the last release.

Merging the release pull request *is* the release: it tags the skill
(e.g. `mtg-deck-publish-v1.2.0`), publishes a GitHub release with the notes,
and writes that skill's `CHANGELOG.md`.

The whole flow:

1. Merge a contribution.
2. release-please opens or updates the release pull request. Nothing to do.
3. When the accumulated changes are worth publishing, merge it. That is the release.

Skills version independently - a fix to one does not release the others.

## What makes it work: commit messages

Version bumps and changelog sections come from
[Conventional Commits](https://www.conventionalcommits.org/). The repository
squash-merges using the **pull request title** as the commit subject, so the title
is what matters, and a CI check rejects titles that do not conform.

| Pull request title | Effect |
| --- | --- |
| `fix(mtg-deck-publish): correct the bulk edit board description` | patch bump, "Fixed" |
| `feat(mtg-deck-publish): add a mulligan section to the primer` | minor bump, "Added" |
| `docs(mtg-deck-publish): clarify the hub selection rules` | no bump on its own; appears under "Documentation" in the next release |
| `docs: clarify the contribution rules` | nothing - repository documentation is outside every package |
| `ci: pin the scanner version` | nothing - hidden, and outside every package |
| `chore: tidy the template` | nothing - hidden from the changelog |

Two things decide whether a title produces a release, and both are easy to get
wrong:

- **Only `fix`, `feat` and a breaking change bump a version.** `docs`, `refactor`,
  `ci` and `chore` are changelog material at most. A run of documentation-only
  pull requests will not open a release pull request by itself.
- **release-please attributes a commit to a package by the files it touches.** A
  change to `CONTRIBUTING.md`, `docs/` or `.github/` is under no package path, so
  it never appears in a skill's changelog whatever its type. Scope the title to a
  skill only when the change is inside that skill's folder.

A breaking change - anything that would surprise someone who already installed the
skill - takes a `!` after the type (`feat(mtg-deck-publish)!: ...`) or a
`BREAKING CHANGE:` footer, and bumps the major version.

If you edit a contributor's title while squashing, that edited title is what counts.

## First release

A `0.0.0` entry in the manifest means *never released*, so release-please does not
derive the first version from conventional commits at all - it proposes the
strategy's initial version, which for `simple` is **1.0.0**. `Release-As:` is not
involved. To start lower, add `"initial-version": "0.1.0"` to the package block in
`release-please-config.json`, then close the open release pull request and let the
next push regenerate it.

## One-time setup

- Settings -> General -> Pull Requests: **Allow squash merging** only, with the
  default commit message set to **Pull request title**.
- Nothing else. release-please runs on `GITHUB_TOKEN`.

A pull request opened by `GITHUB_TOKEN` does not fire the `opened` event for other
workflows, so the release pull request is missing at least one required check:
`pr title` runs on `pull_request_target`, which is never fired for it, so that
check stays permanently pending. The others have been observed reporting normally.
The pull request therefore shows as blocked, and the maintainer merges it with
**"Merge without waiting for requirements to be met"**, which the ruleset bypass
allows. That is expected - its contents are generated from commits that were
already checked. If you would rather every check ran, create a fine-grained
personal access token with contents and pull-requests write, store it as a secret,
and pass it to the action as `token:`.

## What release-please writes

The `simple` release type maintains a `version.txt` in the package folder **only if
one already exists**; none does here, so the release pull request touches just
`.release-please-manifest.json` and the skill's `CHANGELOG.md`. The policy check
exempts `version.txt` anyway, as insurance if one is ever added. Both are
non-markdown or automation-owned files inside a folder
that is otherwise markdown only, so the `Repository policy` check exempts them by
name. If you add a package, keep that exemption list in
`.github/workflows/skill-safety.yml` in step with it.

## Action pinning

Every action is pinned to a commit SHA with the version in a trailing comment.
Dependabot reads that comment to decide what the current version is, so **the
comment has to match the SHA** - a wrong one makes Dependabot "update" you to
something older or stranger than what you have. When pinning by hand, take the SHA
from the release page for the tag you are naming, not from the tip of a branch.
