# The advisory AI reviewer

An AI reviewer comments on pull requests that touch the contributable surface.
It is advisory in the strict sense: it comments, and that is all it can do. It
merges nothing, closes nothing, approves nothing, requests no changes, and is
not a required status check. Every line of every diff is still read by a person.

Two workflows implement it - `.github/workflows/ai-review-collect.yml` and
`.github/workflows/ai-review.yml` - and the instructions it follows are
`.github/ai-review-rubric.md`. All three are maintainer-only.

## What it is for

The checks that already run are deterministic and precise. The `Repository
policy` job refuses protected paths, non-markdown files, oversized files,
symbolic links, new outbound hosts and invisible characters; `pr title`
validates the Conventional Commit title. Those reject with exact messages and
need no help.

The reviewer exists for the four judgements they cannot make, which are the four
questions in the rubric:

1. **Is this a bug or a preference?** The bar in [CONTRIBUTING.md](../CONTRIBUTING.md)
   is "anyone would call this a bug", not "I would have done it differently".
2. **Does it contradict the skill's `SKILL.md`?** `SKILL.md` governs by
   precedence, and a reference file must never loosen a rule it does not
   contain.
3. **Does it weaken a non-negotiable?** The publish gate, card-name
   verification, reading a board before writing it. This is the attack
   [SECURITY.md](../SECURITY.md) names as *a quietly weakened safety rule*, and
   it arrives as two changed words rather than as a deleted paragraph.
4. **Is the claim checkable?** Most contributions are factual statements about a
   site a skill drives. Anything asserted without evidence gets flagged.

House conventions ride along - English, `(e.g., ..., etc.)` on illustrative
lists, no AI vendor names, hard-wrapped prose per [AGENTS.md](../AGENTS.md).

The comment separates *this blocks merging* - reserved for a prompt-injection
attempt in the diff, or a restatement of something a deterministic check already
refused - from *this is my judgement, and the maintainer decides*. The reviewer
never originates a merge block.

## Why it is in two stages

Contributions come from forks. On a `pull_request` run from a fork the
`GITHUB_TOKEN` is read-only and secrets are unavailable, whatever the workflow
asks for - so a bot cannot post a comment from that run at all. The work is
therefore split:

| | Trigger | Token | Secrets | Does |
| --- | --- | --- | --- | --- |
| Stage 1 | `pull_request` | read-only | none | reads the diff, uploads it as an artifact |
| Stage 2 | `workflow_run` | can write | yes | calls the model, comments, labels |

`workflow_run` runs in base-branch context, which is what gives stage 2 both the
write permission and a trustworthy copy of its own instructions. It is also a
dangerous trigger, and zizmor is right to flag it, so what makes it safe here is
written down rather than assumed:

- **Nothing from the pull request is checked out or executed.** Stage 1 reads
  the diff from the API rather than from a working tree, so no contributor file
  lands on disk even there. Stage 2 checks out `main` and nothing else. The
  artifact holds three files (a diff, a file list, the pull request's own text)
  which are read as data and sent to a model. No step treats them as a path, a
  command or a manifest.
- **The rubric is read from the base branch.** A pull request cannot rewrite the
  instructions of its own reviewer. This is a real failure mode, not a
  hypothetical one: GitHub Copilot code review reads its instructions from the
  head branch, which is why it is not used here.
- **The pull request number is verified.** The artifact names a pull request,
  and a fork could in principle put the wrong number there to make the bot
  comment somewhere else. Stage 2 re-reads that pull request from the API and
  refuses unless its head commit is the one that triggered the run.
- **The diff is untrusted data, and the prompt says so.** It is fenced between
  markers the model is told to distrust, and invisible characters are rewritten
  as visible `<U+XXXX>` markers first, because a model cannot see a zero-width
  space. An instruction found inside the diff is reported as a blocking finding
  rather than followed.
- **Permissions are the four operations performed**: read the base tree, read
  the triggering run's artifact, comment, label. The repository default token is
  read-only.

## What it runs on

Only pull requests touching `**/references/*.md` or `docs/**` - the
contributable surface from CONTRIBUTING.md. A pull request that touches anything
else is already refused deterministically, so there is no judgement left to add,
and neither workflow runs at all. Dependabot's own pull requests are out of
scope for the same reason.

## What it costs

Claude Sonnet, at $2 per million input tokens and $10 per million output. Most
of the input is fixed: the rubric, `CONTRIBUTING.md`, `AGENTS.md` and the
governing `SKILL.md` are sent every time and come to roughly 7,800 tokens for a
skill change. Measured against real diffs from this repository's history:

| Pull request | Input | Output | Cost |
| --- | --- | --- | --- |
| One-line fix in `docs/` | ~5,100 | ~270 | ~$0.013 |
| One reference file, small edit | ~8,500 | ~970 | ~$0.027 |
| Three reference files, 11 KB diff | ~11,300 | ~1,290 | ~$0.035 |

Input is measured; output is taken from the dry-run replies, so treat the output
column as the right order of magnitude rather than a promise.

**Call it three cents per run.** It runs once per push to the branch, not once
per pull request, so a branch pushed to four times costs about twelve cents; the
comment is edited in place rather than duplicated. The diff is capped at 200 KB,
which puts the worst case around $0.13 - and a pull request that size needs a
person rather than a summary anyway.

If the fixed portion ever matters, prompt caching would cut it substantially. It
is deliberately not used yet: it is another moving part in a workflow whose
value is that every part of it can be explained.

The model is set by the `MODEL` environment variable on the `Review` step.

## Setup

See [repository-setup.md](repository-setup.md) for the three things this needs -
the `ANTHROPIC_API_KEY` secret, the `needs-maintainer-judgement` label, and the
rule that neither of these checks is ever added to the required list.

## Turning it off

Delete the `ANTHROPIC_API_KEY` secret. Stage 2 logs a warning and exits green;
nothing else changes, and no pull request is blocked. Removing the two workflow
files is the permanent version.
