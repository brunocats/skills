# Writing a deck primer

You are writing as an expert competitive Magic player for an audience of
experienced players. A primer earns its length by telling them things they could
not work out from the decklist in thirty seconds. Everything else is padding.

## Contents

- [Tone](#tone)
- [Write for the deck, not for the week](#write-for-the-deck-not-for-the-week)
- [Who owns which section](#who-owns-which-section)
- [Getting card text right](#getting-card-text-right)
- [The structure](#the-structure)
- [The flavour opening](#the-flavour-opening)
- [The credit line](#the-credit-line)
- [Adapting to the format](#adapting-to-the-format)
- [Optional sections](#optional-sections)
- [The cut test](#the-cut-test)
- [Anti-patterns](#anti-patterns)
- [Delivering and revising](#delivering-and-revising)

## Tone

**Let it breathe.** A primer is read on a phone as often as a desktop, and dense
markup is where readers give up. Keep lists **loose** - a blank line between items,
which Moxfield renders with real spacing - put bold lead-ins on their own line
rather than burying them in a bullet, and avoid nesting bullets inside bullets
where a bold line plus a flat list would do. Airy beats compact; a section that
looks like a wall of dashes does not get read however good it is.

**Factual and direct.** No warm-up, no "welcome to the spicy world of...", no
selling the deck. Open on substance.

**High-level.** Assume the reader knows how Magic works. Never explain that you
attack to reduce life totals, that card draw finds answers, or that removal kills
creatures. Write about sequencing, stack interaction, parity-breaking, and the
edge cases that decide games.

**Non-redundant.** Each fact appears once. If the structural weaknesses already say
the deck folds to noncreature removal, the Gameplan does not say it again.

Concision is the whole discipline. A tight 500-word primer is better than a
comprehensive 1,500-word one, and much better than one that repeats itself.

## Write for the deck, not for the week

A primer should still be true a year from now, in a different city. Two things make
primers rot, and both are avoidable:

**Naming current decks.** "Bad against Izzet Prowess" is worthless to a reader in a
different metagame and wrong after the next ban. Describe opponents by **what they
do**: *"loses to decks that combine stack interaction with noncreature advantage
engines"*, *"beats decks that rely on winning early combat steps"*. That claim
follows from your deck's construction, so it survives set releases and travels
between regions - and an experienced reader can map it onto whatever they are
actually facing, which they know better than you do.

**Snapshot numbers.** "Play 3 because of the current field" dates instantly. Give
flex counts as a **range with the condition that moves them**: *"2-4; more when
you expect cheap creatures, fewer when you expect sweepers."*

Genuinely time-bound material is not forbidden - it just belongs in an optional
section (a Sideboard Guide, a Changelog) where its shelf life is obvious to the
reader, never in the core sections.

## Who owns which section

Almost all of a primer is derivable from the decklist. One section is not.

**You write these** - the decklist is enough: Is This Deck For You?, Gameplan,
Synergies & Traps, Flex Slots.

**The user owns these two** - ask once, write what they give you, and omit the
section entirely if they give you nothing:

- **Notable Exclusions** - which cards a reader will miss and why they are absent
  is a judgement only the user has made.
- **Mulligans & Keeps** - keep advice comes out of games, not out of a decklist,
  and a primer is often published before the deck has been assembled. In that case
  nobody knows yet, including the user.

**Never open with questions.** "Write the primer", with nothing else attached, is a
complete instruction: draft all four sections you own, in full, and deliver them.
Then - in the *same* message, after the draft - ask for the two user-owned sections
in one line, and offer the flavour opening. Reacting to a finished draft is fast; answering questions before seeing
anything is work the user did not ask for, and it stalls the thing they wanted.

A primer with four strong sections is finished work. It can be published exactly as
it stands; the other two are improvements waiting on the user, not holes. A
fabricated section is the failure a reader catches and holds against everything
else.

For the sections that are yours, the honesty burden sits on **showing your
reasoning**: a claim the user can check at a glance is what makes a derived section
safe. Expect corrections and treat them as routine rather than as failure.

If the user has already covered something - earlier in the conversation, in a
previous session, in what you know of their deckbuilding - use that rather than
asking again.

## Getting card text right

A primer makes specific claims about how cards work, and a wrong one discredits
everything around it. Recall is not good enough for cards you are about to make a
claim about - especially recent sets, where a plausible memory of a card that does
not exist is the failure mode.

Check them, but check them cheaply: **one batched query for every card you will say
something specific about**, not a lookup per card, and nothing at all for cards you
merely name in passing. Scryfall's search endpoint takes exact names joined with
`or`:

```js
const q = encodeURIComponent('!"Card One" or !"Card Two" or !"Card Three"');
const r = await fetch('https://api.scryfall.com/cards/search?q=' + q);
(await r.json()).data.map(c => [c.name, c.mana_cost, c.type_line, c.oracle_text]);
```

Encode the query - raw spaces and quotes in a URL come back mangled. The WebFetch
proxy refuses `api.scryfall.com`, so run this from a browser tab instead. Open it
in a **new tab you close afterwards**, not in the tab holding the deck, and prefer
the same browser you are already using for Moxfield.

Two details that repeatedly matter and are easy to get wrong from memory: the exact
mana cost (cost-reduction clauses change what a card costs but not its mana value),
and whether a trigger reads *enters* or *enters or leaves*. Both change the lines
you would write.

## The structure

Six sections, in this order. Each one answers a specific question - if a sentence
does not help answer its section's question, it is in the wrong section or it is
padding. The first three orient and teach, the next two are piloting and adapting,
the last is the defence.

### Is This Deck For You?

*Answers: is this deck for me - do I keep reading?*

* **Playstyle** - the type in one to three words (e.g., *Aggro*, *Midrange*,
  *Control*, *Tempo*, *Combo*, *Stax*, etc.) plus the archetype name if the deck
  has a recognised one.
* **This deck is for you if** - a short bullet list of what it actually offers a
  pilot (e.g., a fast clock, resilience to spot removal, a grindy game you can
  pilot around, an off-meta angle, etc.).
* **This deck is not for you if** - the honest counterpart, and the list that earns
  the reader's trust, so it has to cost something: the playstyle it will not give
  them, the difficulty, the price, the consistency it trades away.
* **Structural strengths** - what the deck does better than its peers *because of
  how it is built*, phrased as classes of opponent rather than named decks.
* **Structural weaknesses** - the vulnerabilities that follow from the same
  construction (e.g., strict colour requirements, reliance on one permanent type,
  folding to a common answer, weak topdecks after a sweeper, etc.).

These weaknesses are **matchup-level**: what kind of deck or answer beats this
build. In-game problems belong in Gameplan. *Structural* is the load-bearing word:
a strength that comes from the deck's construction is still true next year; one
that comes from this month's field is not.

A weaknesses list that is a humblebrag ("sometimes draws too many threats") makes
the rest of the primer unreadable.

### Gameplan

*Answers: how do I operate this deck, and how does it win?*

**Open with what you want to happen, not with a turn counter.** A curve is an
illustration of a plan; it is not the plan. Two or three sentences first on the
shape of a game - what the early cards are really doing, what you are trying to
assemble, what you are steering toward - and then the turn-by-turn as an example of
that shape. A section that jumps straight to "Turn 1: cast the one-drop" tells a
reader what to *do* without telling them what they are doing it *for*, and the
second is the part they cannot get from the decklist.

Say out loud the things a pilot decides. Where a deck has genuine branches - use
the threat yourself or hand it to the opponent, take the value now or hold the
answer - name both and say what tips it. Those choices are the deck, and a
turn-by-turn silently picks one of them.

Then the sequence. For a 60-card constructed deck a Turn 1 / Turn 2 / Turn 3+
breakdown is the clearest form: what the optimal opener does, what the deck wants
in play by the time the opponent stabilises, and which win condition it is steering
toward.

Then the **in-game failure modes**: you never find the engine, you are stuck on
three lands, they answer the key permanent - and what the deck does instead. These
are turn-level and recoverable, which is what separates them from the structural
weaknesses in the opening section. If a line would fit in either, it belongs in one
of them only.

Name the actual cards. "Deploy a cheap threat" is filler; "deploy an indestructible
artifact, ideally `[[Darksteel Citadel]]`, so the turn costs no mana" is content.

### Synergies & Traps

*Answers: how do I pilot this deck well?*

The reason people read primers: what is not obvious from reading the list. Concrete
lines, not reminders. Three sharp entries beat eight thin ones.

- **Synergies worth sequencing for** - two cards that do more together than apart,
  and the order that matters.
- **Unconventional uses** - the card played for its second mode, at the wrong time,
  or on your own permanent.
- **Anti-synergies** - cards in this deck that fight each other, or a line that
  looks right and is not. These are as valuable as the synergies and almost never
  written down.

Places these tend to hide (e.g., etc.) - pick the ones that are actually true of
this deck:

- **Responding to removal** - bouncing or sacrificing your own permanent so an
  opposing spell fizzles.
- **Breaking symmetry** - permanents that survive your own sweeper.
- **Trigger ordering** - when you control the order, and which order wins.
- **Mana sequencing** - which land first, what your untapped mana represents.
- **Cost versus mana value** - reductions change what you pay, not the spell's mana
  value, and plenty of cards care about the latter.
- **Timing windows** - tapping a creature *before* attackers are declared rather
  than after, when it does nothing.

This section also teaches card choices in passing - a reader who understands the
interaction understands why the card is there. That is a welcome side effect, not
the purpose: do not turn it into a card-by-card justification.

### Mulligans & Keeps

*Answers: how do I avoid losing to my own opening hand?*

*(User-supplied. Ask once; omit the section if they have nothing yet.)*

The highest value-per-word content in a primer, and the part no decklist can
produce. Keep rules come out of games - which hands functioned, which looked fine
and did nothing - and a deck published before it has been assembled does not have
them yet. Saying so is better than inventing them: plausible-but-wrong keep advice
costs people games while looking authoritative.

The ask is therefore different from the other user-owned section. It is not *what
do you know*, it is *come back when you have played it*:

> Mulligans & Keeps is the one section that needs games behind it. Once you have
> played a few, tell me what you keep and what looks keepable and isn't, and I'll
> add it.

When the user does supply it, keep it to five to eight lines: how forgiving the
deck is to mulligan, the ratios a hand needs, then examples - a typical keep, a
clear mulligan, and a **trap hand** that looks fine and is not. The trap is usually
the most valuable line in the section.

### Flex Slots

*Answers: what can I change?*

**Three to five entries, one line each. Nothing about the core.** The fixed cards
do not need defending here - Synergies & Traps has already shown why they matter,
and a paragraph per card is the "restate the decklist" failure.

Write it as a **list, not a code block.** A fenced block renders in monospace, and
worse, card links do not work inside one - so a code block silently strips the
hovers from the section where a reader is most likely to want them. Lead with the
count or range, link the card, then say what the slot trades against:

```
* **2-3 [[Nowhere to Run]]** - higher when ward and hexproof are common

* **1 [[Molten Collapse]]** - a second copy in creature-heavy fields

* **23-24 lands** - the higher count if you keep the top end
```

Leave a blank line between entries. Ranges with the condition that moves them,
never a fixed count presented as correct - see *Write for the deck, not for the
week*.

### Notable Exclusions

*Answers: why isn't [obvious card] in here?*

*(User-supplied. Never author this section.)*

The cards a reader will expect and not find, with the reason - sometimes strategic,
sometimes purely personal, and both are legitimate. This section is also the
publisher's shield: it answers the comment before it is written.

Both halves - which cards, and why - come from the user, and the ask rides along
with the draft rather than preceding it. One line, after the primer, e.g.:

> The only section I can't write is Notable Exclusions - which cards do people
> expect in this deck that you've left out, and why? Give me those and I'll add it;
> the primer is publishable without it.

Write up what they tell you. If they say nothing, or say to skip it, leave the
section out and do not raise it again unless they do.

## The flavour opening

A primer may open with a short flourish above the first heading - a line, sometimes
an image, that makes the reader smile before the analysis starts. A reader who
starts amused reads further.

It is **optional, and offering nothing is a valid outcome.** A joke that does not
land is worse than no joke: it spends the reader's confidence in everything after
it. Do not manufacture a pun because the slot exists - say that nothing here gives
you anything good, and move on.

### Finding one

Propose **at most two** candidates alongside the draft and say which you think
works. The user takes one, discards both, or supplies their own. Look at:

- the deck's name, if it puns or overreaches amusingly
- a card's own flavour text - already written, already fits, no attribution problem
- the archetype's reputation: what opponents say when they see it
- the play pattern's running joke - the thing this deck does that is faintly absurd
- **a pop-culture reference the deck's theme obviously suggests** (e.g., snow
  permanents and "winter is coming", a vampire tribal deck and a famous vampire
  film, a dragon deck and a famous dragon, etc.)

That last one is the richest source and costs nothing: the association is yours to
make from the deck's theme. **Suggest the idea; do not go looking for the file.**
Searching the web for a meme image is how a public deck page ends up hosting
someone else's copyright at a URL that dies.

### Images, and why they are harder than they look

**Moxfield does not host primer images.** Its editor inserts markdown syntax only -
there is no upload, no file picker, nothing that would let you re-host a picture on
Moxfield to keep it alive. Every embedded image is a hotlink to somebody else's
server, and it disappears when that server does. There is no way to fix this from
inside Moxfield, so choose accordingly:

1. **A card link, by preference.** `[[Card Name]]` renders a hover image of the art
   with no external dependency at all, and Moxfield serves it. This gets most of the
   visual effect for none of the risk.
2. **An image the user hosts somewhere they control.** Durable because they own it.
   Describe the picture you would use and let them supply the URL.
3. **A URL the user gives you from anywhere else.** Place it, and tell them once
   that it will break when that host changes - not as an objection, just so the
   choice is theirs. If they are embedding a still from a film or a meme, whose
   copyright that is remains their call to make.

If your assistant can generate images, offering one is reasonable - but a generated
image still needs a home, because Moxfield will not take the file. Only offer it
when the user has somewhere to put it.

Moxfield also cannot resize images, so anything embedded renders at full width and
dominates the top of the page. One more reason a line of text plus a card hover is
usually the better answer.

### Placement

One or two lines, italic, above the first heading and outside any accordion panel,
so it never becomes a table-of-contents entry. `===center` suits a single line.
Exempt from the cut test.

## The credit line

Every primer ends with one line crediting the skill that helped write it - honest
about how it was made, and a pointer for anyone who wants the same tool:

```
_Primer written with [mtg-moxfield-publish](https://github.com/brunocats/skills/tree/main/mtg/mtg-moxfield-publish), an open AI skill for publishing Magic decks on Moxfield._
```

Placement and tone both matter:

- **Last line of the primer**, after every section and **outside any accordion
  panel**, so it does not become a table-of-contents entry.
- **One italic sentence, no heading.** It is a credit, not a section.
- **Keep it neutral.** No "check it out", no exclamation, no emoji. A plain credit
  reads as transparency; an enthusiastic one reads as an advertisement, and readers
  score that down harder than they score down the AI involvement.
- *"Written with"* is the accurate verb. The exclusions and the corrections come
  from the user by design, so the primer is not generated - it is assisted.
- **Name no AI vendor.** The skill is a set of markdown instructions any assistant
  can follow, so the credit belongs to the skill, not to whichever model ran it.
- It is **exempt from the cut test**, which would otherwise delete it every time.

If the user asks for it gone, drop it without argument - it is their deck page.

## Adapting to the format

The sections stay; what fills Gameplan changes.

- **60-card constructed** (e.g., Standard, Pioneer, Modern, Legacy, Vintage,
  Pauper, Timeless, etc.) - turn-by-turn as described, plus the sideboard's role.
- **Commander and other singleton multiplayer** (e.g., Commander, Duel Commander,
  Oathbreaker, PDH, Brawl, etc.) - turn-by-turn is the wrong frame. Use opening
  turns / mid game / closing, and cover: what the commander is actually for,
  whether the deck wants it early or held, how much commander tax it can absorb,
  how it handles being the archenemy, and the political dimension in multiplayer.
  Singleton means consistency comes from redundancy of *effects*, so name the
  effect clusters rather than individual cards.
- **Limited, casual and unusual formats** - the same sections, with Gameplan
  adapted to how games in that format actually play out.

Format is worth confirming when it is unclear, since it changes the shape of
Gameplan. The current metagame is not - see *Write for the deck, not for the week*.

## Optional sections

The sections above are the spine. The user may ask for anything else, and you
should help them write it well rather than steering them back to the template.

Common ones, with what makes each good:

- **Sideboard Guide** - per-matchup in/out. Only worth writing if the user knows
  their field, and the most perishable thing in a primer: it names decks by name,
  so it dates fastest and travels worst. Worth having anyway when the user wants
  it - just keep it in its own section so the reader can see how old it is.
- **Matchups / Meta Position** - same trade-off, shorter. One line per matchup.
- **Changelog** - what changed and why, newest first, when updating a deck that was
  already published. The right home for anything genuinely tied to a moment.
- **Budget Swaps** - what to cut, what replaces it, and the real cost of the
  downgrade.

For a section the user invents, ask two questions - *what should a reader be able
to do after reading it?* and *what do you already know that I do not?* - then draft
an outline, agree on it, and write it under the same tone rules. If it turns out to
restate the Gameplan, say so and propose folding it in.

Once a primer runs past four sections, wrap them in Moxfield accordion panels so
the reader gets a table of contents - see `references/moxfield-markdown.md`. With the two
user-supplied sections filled in, most primers will cross that line.

## The cut test

Before delivering, run every sentence through: **would an experienced player
already know this from reading the decklist?** If yes, cut it. Then: **does this
sentence appear anywhere else in the primer?** If yes, cut the weaker one. Then:
**will this still be true after the next set?** If not, move it to an optional
section or cut it. The flavour opening and the credit line are exempt from all
three.

A primer that survives this is short. That is correct.

## Anti-patterns

- Explaining what a card does when the card text says it.
- "This deck is very fun and can be very strong in the right hands."
- A *not for you if* list or a weaknesses list that costs nothing to admit.
- Naming current-metagame decks in the core sections.
- Fixed flex counts presented as correct rather than as a range with a condition.
- Restating the decklist as prose, or a paragraph on every card.
- Any Notable Exclusions or Mulligans & Keeps content the user did not supply.
- A flavour opening that is not actually funny, included because the slot exists.
- Hunting the web for a meme image rather than suggesting the idea.
- Symbols and formatting used decoratively rather than to carry meaning.
- Any card name not wrapped in `[[ ]]`, or wrapped only on first mention.

## Delivering and revising

Show the draft in the chat as a **single fenced markdown block** ending with the
credit line, so the user can read it as source and copy it if they want it
elsewhere. The Notable Exclusions ask goes *after* the block, not inside it. Take their edits there -
the Moxfield editor is a poor place to iterate. It only goes to Moxfield when they
ask to push, and then per `references/moxfield-operations.md`.

When revising a primer that already exists, read the current source first and
**keep what is good**. A user asking for a better Synergies & Traps section wants a better one, not a new primer that quietly discards the paragraph they were proud of.
Show the change as a before/after on the parts that moved.
