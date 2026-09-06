# Repository setup

Some of the controls this repository relies on are GitHub's own and have to be
configured once, in the repository settings. The rest live in
`.github/workflows/skill-safety.yml`. This page covers both, and why the split
falls where it does.

## Ruleset on `main`

Settings -> Rules -> Rulesets -> **New branch ruleset**, targeting `main`,
enforcement **Active**, with the repository owner in **Bypass list** (so the
maintainer can still edit protected paths directly).

Enable:

- **Require a pull request before merging.** Nothing reaches `main` without a diff
  to read. Set **Required approvals** to **1** *and* tick **Require review from
  Code Owners**. Both, not either: the code-owner requirement is silently inert
  while required approvals is 0, which is recorded at the top of
  [`.github/CODEOWNERS`](../.github/CODEOWNERS). The owner is in the bypass list,
  so this gates contributors and costs the maintainer nothing.
- **Require status checks to pass** - select `Agent Skill scan`, `Repository
  policy`, `zizmor` and `pr title`. Each of those is the *job* name, which is what
  a check reports as; the workflow name is not.
- **Block force pushes.**
- **Restrict deletions.**

## Why the path rules are in CI and not in the ruleset

The obvious place for "no `SKILL.md` changes, markdown only, nothing over 1 MB"
is a ruleset. It is not available here. **Restrict file paths**, **Restrict file
extensions** and **Restrict file size** are *push* ruleset rules, and [GitHub's
documentation](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
is explicit: "Push rulesets are available for the GitHub Team plan in internal and
private repositories, and forks of repositories that have push rulesets enabled."
This is a public repository on a personal account, so those three rules cannot be
turned on at all - they do not even appear in the branch ruleset UI the rest of
this page sends you to.

So they are implemented in the `policy` job of `skill-safety.yml` and made a
required status check. That is weaker than a ruleset in one specific way - a
required check is enforced by branch protection, which the owner bypasses - and
equivalent in the way that matters against a contributor:

For `pull_request`, GitHub takes the *workflow file* from the base branch, but
`actions/checkout` checks out the **merge commit** - which contains the
contributor's changes. Any checker script living in the repository would therefore
be the contributor's copy of that script. A pull request could rewrite the checker
in the same change the checker is meant to block. The workflows here avoid this by
keeping their logic inline in the workflow file, and by reading no manifest or
requirements file from disk - which is why the scanner's version is written inline
and bumped by hand.

## The advisory AI reviewer

Three things, and the third is the one that matters.

**A secret.** Settings -> Secrets and variables -> Actions -> **New repository
secret**, named `ANTHROPIC_API_KEY`. Without it stage 2 logs a warning and exits
green, so the reviewer is off rather than broken - which is also how to turn it
off later.

**A label.** Create `needs-maintainer-judgement` (Issues -> Labels -> **New
label**). The workflow applies and removes it but never creates it, so that it
needs no permission to write labels repository-wide.

**Not a required check.** Neither `ai review` nor `ai review (collect)` is ever
added to the required status checks above. That list stays at four: `Agent Skill
scan`, `Repository policy`, `zizmor` and `pr title`. `ai review` may go red -
deliberately, so a contributor sees a finding without opening the comment - but
red there is a signal, not a gate. Two reasons it has to stay that way. It
depends on a third-party API and a secret, so an outage or an expired key would
block every merge in the repository. And it is a model's opinion, which is not
the kind of thing this repository lets refuse a change; only deterministic
checks do that.

[ai-review.md](ai-review.md) covers what the reviewer does, why it is split
across two workflows, and what it costs.

## Private vulnerability reporting

Settings -> Security -> **Private vulnerability reporting**: enable it. SECURITY.md
tells reporters to open a private advisory instead of a public pull request, and
without this switched on there is no button for them to press - so they will open
a public issue instead, which is the outcome the policy exists to avoid.

## Merge strategy

Settings -> General -> Pull Requests: allow **squash merging only**, with the
default commit message set to **Pull request title**. Release automation reads
those titles - see [releasing.md](releasing.md).

## Verify it before announcing the repository

First, re-read one number in the ruleset: **Required approvals**. A ticked "Require
review from Code Owners" next to a 0 there looks like a configured gate and is not
one, and nothing in CI will tell you. It is the only control on this page that can
fail silently while appearing correct.

Then three behaviours that cannot be confirmed from the settings pages at all,
all of which decide whether outside contributions can merge at all or get
reviewed. Open one throwaway pull request **from a fork** and check:

- The `Agent Skill scan` job passes. On a fork pull request the `GITHUB_TOKEN` is
  read-only whatever the workflow requests, so the SARIF upload is skipped by
  condition rather than allowed to fail the job.
- The `pr title` check appears and reports against the pull request's head commit.
  It runs on `pull_request_target`, which is necessary for the action to read a
  fork's title at all.
- The `zizmor` job **fails** on a finding. `advanced-security: false` is set for
  exactly that reason, but the action's documentation only states that the
  Advanced Security mode does *not* fail - it does not state the inverse. Confirm
  it by temporarily unpinning an action on the test branch and checking the job
  goes red.
- The advisory reviewer comments. Its two-stage split exists precisely because a
  fork's token cannot write, so a pull request from a branch of this repository
  proves nothing about it - only a fork does. See [ai-review.md](ai-review.md).
