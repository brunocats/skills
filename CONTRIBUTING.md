# Contributing

Thanks for improving a skill. Three things to know before you start.

## Pull requests, not issues

Send a pull request with the change already made. I review pull requests; I do not
work through issues and implement suggestions, so an issue describing a problem
will most likely sit there. If you can describe the fix, you can make it - these
are markdown files.

You do not need the repository checked out to do this. If you are running one of
these skills with an assistant, ask it to fork, make the change and open the pull
request.

## Quality fixes, not preferences

The bar is **"anyone would call this a bug"**, not "I would have done it
differently".

**Welcome:**

- A factual error - a page, control or behaviour described wrongly
- Something that has moved or changed on a site a skill drives
- An instruction that does not work, or is ambiguous enough to be followed wrongly
- Reasoning a skill gets wrong in a way that would mislead any user
- A gap where a skill has nothing to say and clearly should

**Not welcome, and I will close them:**

- Tone, register, naming taste, section preferences
- "I'd rather it asked me about X" / "I'd rather it didn't"
- Anything that weakens a safety rule

If your change is about how *you* like the output, keep it in your own copy. That
is what a skill's local learning loop is for.

## What you may change

Skills run with the user's browser and accounts, so the contributable surface is
deliberately narrow. [SECURITY.md](SECURITY.md) explains why.

**Open to pull requests:** `references/*.md` in any skill, and the documentation at
the repository root.

**Maintainer-only, and blocked by a repository ruleset:** any `SKILL.md`, any
skill's `feedback/LEARNINGS.md`, anything under `.github/`, any non-markdown file,
and any new link to a host outside the allowlist. These are refused by GitHub
itself, not by a reviewer's judgement, so a pull request touching them cannot be
merged. If you believe such a change is genuinely needed, open a pull request
against a reference file making the case, and say so plainly.

## Pull request titles

The changelog and version numbers are generated from commit messages, and this
repository squash-merges using the pull request title. So the title has to be a
[Conventional Commit](https://www.conventionalcommits.org/), and a check will fail
if it is not:

```
fix(mtg-moxfield-publish): correct the bulk edit board description
feat(mtg-moxfield-publish): add a mulligan section to the primer
docs: clarify the contribution rules
```

Use `fix` for a correction, `feat` for something the skill could not do before,
`docs` for documentation. The scope is the skill's name, omitted for
repository-wide changes.

## What a mergeable pull request looks like

- One change, one concern, one file wherever possible
- Markdown only
- Matches the surrounding file - English, illustrative lists marked
  `(e.g., ..., etc.)`, no AI vendor names, same register
- The description fills in the template, including why it is not a preference

Automated checks run on every pull request - an Agent Skill scanner, a workflow
audit and the repository policy check. They are a floor, not a verdict: every line
still gets read.
