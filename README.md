# skills

AI skills, organised by domain. Each skill is a folder: a `SKILL.md` with the
workflow, and `references/` with the detail it loads on demand. They are plain
markdown, so any assistant that can read a skill folder can run them.

```
mtg/     Magic: The Gathering
```

## Skills

| Skill | What it does |
| --- | --- |
| [mtg/mtg-moxfield-publish](mtg/mtg-moxfield-publish/) | Runs an interactive session to publish or update a Magic: The Gathering decklist on Moxfield - deck name, description, primer, hubs, image and visibility - and pushes the approved changes by driving a browser. |

## Using one

Take the folder from a [release](https://github.com/brunocats/skills/releases)
rather than from `main`, drop it wherever your assistant loads skills from, and ask
for the thing it does. Each skill is versioned and tagged separately
(`mtg-moxfield-publish-v1.2.0`) and carries its own `CHANGELOG.md`.

## Is this safe to install?

A skill runs with your browser and your accounts, so you should read one before you
install it - including these. [SECURITY.md](SECURITY.md) sets out what a malicious
change to a skill could do, the controls that exist to stop one reaching you, and
what is deliberately not claimed.

## Contributing

Improvements to skill *quality* are welcome as pull requests. Personal preferences
are not - see [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT. See [LICENSE](LICENSE).
