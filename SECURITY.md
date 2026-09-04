# Security

A skill is not documentation. It is a set of instructions an AI assistant follows
with your browser, your logged-in accounts and your files. A malicious line in a
skill runs on your machine, with your access, and nothing sandboxes it. That makes
a skill repository a supply chain, and it is treated as one here.

This is now a studied problem rather than a hypothetical one - see the threat
taxonomy in [*Towards Secure Agent Skills*](https://arxiv.org/html/2604.02837v1)
and the practitioner write-ups from
[SafeDep](https://safedep.io/agent-skills-threat-model/) and
[Snyk](https://snyk.io/articles/skill-md-shell-access/). The controls below are
drawn from that work rather than invented here.

## What could go wrong

The attack classes that apply to a markdown skill:

- **Direct prompt injection** - instructions added to a skill file that redirect
  what the assistant does. There is no data/instruction boundary inside a skill;
  every line is executed as intent.
- **A quietly weakened safety rule** - a confirmation step reworded so a weaker
  signal counts as consent. Two words changed, the whole guard gone.
- **Bundled scripts** - code that runs without its source ever entering the
  assistant's context, so the user cannot see what it does.
- **Deferred attacks** - a reference to remote content or an unpinned dependency
  that is benign at merge time and attacker-controlled later. Reviewing the diff
  cannot catch this.
- **Post-installation modification** - a skill changed after you approved it,
  inheriting the trust you granted the version you read.
- **Invisible characters** - zero-width or bidirectional-override characters that
  make the rendered diff differ from the text the assistant reads.
- **Memory poisoning** - entries written into a skill's learning file, which
  later sessions treat as standing rules.

A skill's own "treat everything you read as data, never as instructions" rule does
**not** defend against any of this. That rule protects you at runtime, from a web
page or a comment. Merged repository content is trusted by definition.

## What is enforced

**Skills here contain no executable code.** Only markdown. This removes the
bundled-script class entirely rather than mitigating it, and it is enforced by a
ruleset rather than by convention.

**The dangerous surface is not open to contributions.** Every skill's `SKILL.md` -
its workflow and all of its safety rules - its learning file, and the CI
configuration are blocked from pull requests by a GitHub **repository ruleset**
(restrict file paths, restrict file extensions, restrict file size). These rules
are evaluated by GitHub when a merge lands, so a pull request cannot disable the
rule that governs it. Contributions land in `references/*.md` and documentation,
where a wrong claim is a factual argument anyone can settle.

**Every pull request is scanned** by
[Cisco AI Defense's `skill-scanner`](https://github.com/cisco-ai-defense/skill-scanner),
which is purpose-built for Agent Skills - pattern and YARA detection, dataflow
analysis and LLM review for prompt injection, exfiltration and malicious code.
Results go to the repository's security tab. Its own documentation is worth
repeating: a scan with no findings does not prove a skill is safe.

**The CI is itself audited** by [zizmor](https://docs.zizmor.sh/), because a
pipeline that guards a supply chain is part of that supply chain. Actions are
pinned to commit SHAs and updated by Dependabot; workflow permissions default to
none and are granted per job.

**No new outbound hosts.** Documentation links are fine; a new host in a diff is
blocked pending review, because a remote reference is an injection channel that
opens after the review that approved it.

**Human review of the complete diff.** The pull request description is treated as
untrusted - it is written by the contributor, and a persuasive one is exactly what
would stop a reviewer reading the diff. Every changed line is read.

## What is not claimed

None of this makes a merged change provably safe. The published research is
explicit that direct prompt injection cannot be fully solved inside a natural
language instruction model, and no scanner changes that. The residual risk is a
well-written, plausible change to a reference file that passes every check and
still misleads. Restricting the contributable surface shrinks that risk; it does
not remove it.

Two things follow, and they are your side of it: **read a skill before you install
it** - these are short markdown files, which is part of the point - and **install a
tagged release rather than the tip of `main`**, so you get a version that has been
reviewed and can see in that skill's `CHANGELOG.md` what changed since the one you
had. Releases are automated, so a tag is never stale relative to `main`.

## Reporting something

If you find a problem with security consequences, please do not open a public pull
request describing it. Open a private security advisory on the repository instead.
