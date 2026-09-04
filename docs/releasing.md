# Releasing

Releases are automated. The maintainer merges pull requests; version numbers,
changelogs, tags and GitHub releases happen without intervention.

## How it works

[release-please](https://github.com/googleapis/release-please) reads the commit
history on `main` and keeps a standing **release pull request** up to date. That
pull request holds the next version number and the changelog entries generated
from the commits since the last release.

Merging the release pull request *is* the release: it tags the skill
(e.g. `mtg-moxfield-publish-v1.2.0`), publishes a GitHub release with the notes,
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
| `fix(mtg-moxfield-publish): correct the bulk edit board description` | patch bump, "Fixed" |
| `feat(mtg-moxfield-publish): add a mulligan section to the primer` | minor bump, "Added" |
| `docs(mtg-moxfield-publish): clarify the hub selection rules` | no bump on its own; appears under "Documentation" in the next release |
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
skill - takes a `!` after the type (`feat(mtg-moxfield-publish)!: ...`) or a
`BREAKING CHANGE:` footer, and bumps the major version.

If you edit a contributor's title while squashing, that edited title is what counts.

## First release

The manifest is seeded at `0.0.0`, so the first `feat:` produces `0.1.0`. To go
straight to `1.0.0`, put `Release-As: 1.0.0` in the body of any commit before
merging the release pull request.

## One-time setup

- Settings -> General -> Pull Requests: **Allow squash merging** only, with the
  default commit message set to **Pull request title**.
- Nothing else. release-please runs on `GITHUB_TOKEN`.

Pull requests opened by `GITHUB_TOKEN` do not trigger other workflows, so the
safety checks do not run on the release pull request itself. That is expected - its
contents are generated from commits that were already checked - and the maintainer
is in the ruleset bypass list, so required checks do not block merging it. If you
would rather they ran, create a fine-grained personal access token with contents
and pull-requests write, store it as a secret, and pass it to the action as `token:`.

## What release-please writes

The `simple` release type maintains a `version.txt` alongside `CHANGELOG.md` in the
skill's folder. Both are non-markdown or automation-owned files inside a folder
that is otherwise markdown only, so the `Repository policy` check exempts them by
name. If you add a package, keep that exemption list in
`.github/workflows/skill-safety.yml` in step with it.

## Action pinning

Every action is pinned to a commit SHA with the version in a trailing comment.
Dependabot reads that comment to decide what the current version is, so **the
comment has to match the SHA** - a wrong one makes Dependabot "update" you to
something older or stranger than what you have. When pinning by hand, take the SHA
from the release page for the tag you are naming, not from the tip of a branch.
