# Repository setup

The controls this repository relies on are mostly GitHub's own. They have to be
configured once, in the repository settings - they are not in the codebase, which
is the point: a control that lives in a file the pull request can edit is not a
control.

## Ruleset on `main`

Settings -> Rules -> Rulesets -> New branch ruleset, targeting `main`,
enforcement **Active**, with the repository owner in **Bypass list** (so the
maintainer can still edit protected paths directly).

Enable:

- **Require a pull request before merging.** Nothing reaches `main` without a diff
  to read.
- **Require status checks to pass** - select `Agent Skill scan`, `Repository
  policy`, `zizmor` and `pr title`.
- **Restrict file paths.** Block:
  - `*/*/SKILL.md` - each skill's workflow and safety rules
  - `*/*/feedback/**` - skill learning files, which are user state
  - `.github/**` - the CI that enforces everything else
- **Restrict file extensions.** Block anything executable that could be bundled
  with a skill (e.g., `.sh`, `.py`, `.js`, `.rb`, `.pl`, `.ps1`, `.bat`, `.exe`,
  `.dll`, `.so`, `.wasm`, etc.).
- **Restrict file size** - a small cap, e.g. 1 MB. Skills are markdown.

Path and extension rules are evaluated by GitHub when the merge lands on `main`,
so a pull request cannot disable the rule that governs it. This is the difference
between these and any check that runs from the repository's own files.

## Why not enforce this in CI instead

For `pull_request`, GitHub takes the *workflow file* from the base branch, but
`actions/checkout` checks out the **merge commit** - which contains the
contributor's changes. Any checker script living in the repository would therefore
be the contributor's copy of that script. A pull request could rewrite the checker
in the same change the checker is meant to block.

The workflows here avoid this by keeping their logic inline in the workflow file
rather than in a script on disk, but the structural fix is the ruleset.

## Merge strategy

Settings -> General -> Pull Requests: allow **squash merging only**, with the
default commit message set to **Pull request title**. Release automation reads
those titles - see [releasing.md](releasing.md).
