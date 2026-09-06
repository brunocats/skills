# Naming a deck

A deck name does two jobs at once: it tells a stranger scrolling a
list of decks what this is, and it gives the user something they are happy to say
out loud. A name that only does the second is a private joke; a name that only
does the first is a filing code.

## What the name has to earn

- **Identifiable** - a reader who plays the format recognises the deck from the
  name alone.
- **Searchable** - it contains at least one term people actually type: the guild
  or colour, the payoff card, or the established archetype word.
- **Honest** - it does not promise colours, a commander, or an archetype the list
  does not deliver.
- **Short** - two to five words. Long names get truncated in deck lists and on
  deckbox images.

A site may impose its own content rules on names - Moxfield asks that they stay
family-friendly, e.g. - and those are rules, not suggestions.

## Work out what the deck actually is

Before generating anything, settle three facts from the list:

1. **Colour identity** - and its shorthand (e.g., Dimir (UB), Jund (BRG), Mono Red).
2. **The engine or payoff** - the one to three cards the deck is built *around*.
   These are the cards with the most copies, the most synergy edges to the rest of
   the list, or the ones the mana base and the removal suite are bending to
   support. Not the most expensive card, and not the commander by default.
3. **The archetype word** - what a player would call this: Affinity, Tempo,
   Reanimator, Stax, Voltron, Death & Taxes. If an established archetype name fits
   the deck, it is usually the strongest anchor available.

## Patterns that work

| Pattern | Example | When |
| --- | --- | --- |
| Colour + payoff | Dimir Ensoul | The payoff card *is* the deck |
| Colour + archetype | Izzet Tempo, Mono-Red Aggro | Standard, recognisable shell |
| Payoff + archetype | Ensoul Affinity | Payoff needs the archetype for context |
| Established archetype alone | Affinity, Death's Shadow | The deck is a known thing |
| Flavour, then anchor | Ensouled Machine // Dimir Affinity | User wants character without losing searchability |
| Commander + theme | Muldrotha Value, Krenko Tokens | Commander and similar formats |

The `Flavour // Anchor` shape is the useful compromise when the user wants a name
with personality: the evocative half carries the character, the half after the
separator keeps the deck findable.

## What to avoid

- Generic (e.g., *My Deck*, *Pioneer Deck v3*, *Test List*). Version numbers belong
  in the description or the changelog, not the name.
- Pure inside joke with no anchor - nobody finds it, and the user cannot explain
  it to an opponent in one sentence.
- Misleading: a name implying colours or an archetype the list does not play.
- Colliding with a famous unrelated archetype, which sends the wrong readers.
- Padding words (e.g., *Ultimate*, *Optimized*, *Competitive*, *Budget-Friendly*,
  *2.0*, etc.).

## How to propose

Offer **four to six** candidates spread across registers - literal and searchable,
archetype shorthand, and one or two with more character - each with a one-line
rationale saying what it anchors on. Then recommend one and say why, briefly.

The point of the spread is to find the user's *register*, not to win with one
option. If they reject a name, the useful next question is which direction was
wrong - too plain, too clever, wrong archetype word - and generate again in that
direction rather than defending the shortlist.

Sample shape:

```
Dimir Ensoul            plainest; colour + payoff, instantly identifiable
Ensoul Affinity         uses the archetype word people browse for
Steel Tempo             leads with the play pattern rather than the card
Ensouled Machine        flavour; "Ensoul" still anchors the search
Blade of the Citadel    evocative; weakest for search
```

Literal and searchable first, character later - that ordering is the
recommendation, not just the sample's accident.

If the user says a name "doesn't mean anything", that is feedback about register,
not about that one name - handle it per `references/feedback-loop.md` so it sticks.

## The description goes with it

Writing the description is the step straight after the name, and what it has to
be is set by the site: how long it may run, where it renders, and what the page
already shows around it. `references/sites/moxfield/description.md` for Moxfield.
