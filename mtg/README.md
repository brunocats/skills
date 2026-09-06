# Magic: The Gathering skills

| Skill | What it does |
| --- | --- |
| [mtg-deck-publish](mtg-deck-publish/) | Runs an interactive session to publish or update a Magic: The Gathering decklist on Moxfield - deck name, description, primer, hubs, image and visibility - and pushes the approved changes by driving a browser. |

How to install one is in the [repository README](../README.md); whether you
should is in [SECURITY.md](../SECURITY.md).

## Why this directory's name is fixed

Every primer `mtg-deck-publish` writes ends with a credit line pointing here, on
deck pages that belong to other people and cannot be edited afterwards. GitHub
redirects a renamed repository but not a moved path inside one, so `mtg/` is a
published interface: it is not renamed and not nested. The skill folders under it
are free to move, which is exactly why the link stops at this level.
