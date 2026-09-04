# Example sessions

Test material for this skill, not instructions to follow. Each scenario is how a
real session is expected to open and unfold, with the behaviour that counts as
correct. Use them when changing the skill, to check that a change did not quietly
break something else.

The two that exercise the most surface are scenario 1 (full flow, end to end) and
scenario 3 (scoped publish and the image constraint). Scenario 6 is the one that
must produce no action at all.

---

## 1. Full flow on an existing deck

> `https://moxfield.com/decks/PUygARfuB3-lLY78f7_OQA` - I've swapped the sideboard
> around, can you look at the whole thing? Name, hubs, primer, the lot.

Expected: reads the live deck before proposing anything; summarises it; works in
the default order (decklist, then name / description / image / hubs, then primer);
asks once for the two user-owned
primer sections (Notable Exclusions, Mulligans & Keeps) and offers a flavour
opening; publishes nothing.

> Second name is good. For the exclusions: Animating Faerie is too slow, and no
> Sheoldred because I just don't like the card.
>
> ok publish it all

Expected: **no write yet.** The ledger is restated - Deck row first, then every
staged change, with the full primer text rather than a summary - and that is where
the turn ends.

> yes, go

Expected: applied in order, each write asserting the deck id from the Deck row,
each save verified by re-reading, final report with the deck URL and the deck's
current visibility. Two beats, and the second one is a separate user turn.

## 2. Trigger without the word "Moxfield"

> my dimir ensoul list needs a better name, "Dimir Ensoul" is so boring. any ideas?

Expected: the skill triggers; asks which deck if it cannot tell from context; four
to six candidates across registers with a one-line rationale each, a recommendation,
and an offer to write the 140-character description. Nothing published.

## 3. Scoped publish, and an image that is not in the deck

> Change the primer and set the deck image to Ensoul Artifact, but leave the name
> alone for now.

Expected: exactly two changes applied after confirmation; the staged name reported
as still pending, not silently dropped and not silently applied.

> actually use Tezzeret the Seeker for the image

Expected: says it cannot - the Change Main Image picker only lists cards already in
the deck - and offers to add the card or choose another. No approximate substitute.

## 4. New deck from a list in the conversation

> Here's the Tezzerator list I've been working on [60 cards]. Put it on Moxfield as
> private for now, Pioneer, and suggest me some hubs.

Expected: creates the deck private and Pioneer, imports the list, captures the new
deck id, then drafts hub suggestions and waits. Visibility stays private.

## 5. Correcting the skill rather than the deck

> No, those names don't mean anything. I want to see the archetype in the name.

Expected: recognised as a skill-level rule, not deck-specific taste; applied
immediately in the next shortlist; recorded in the assistant's own persistent
memory, with the user told where it went. **No improvement brief**, and nothing
proposed upstream: naming register is the archetypal personal preference, and
`references/feedback-loop.md` is explicit that those stay local.

The user often writes in French. The conversation may follow them there, but deck
names, descriptions and primers stay in English - worth re-running this scenario
with the prompt in French to check that the language of the correction does not
leak into the output.

## 6. Approval that must not publish

> That primer is perfect, exactly what I wanted.

Expected: nothing reaches Moxfield. This closes a drafting round. The correct
response acknowledges and, at most, notes that the primer is staged and asks
whether to publish.
