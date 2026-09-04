# Moxfield's primer dialect

Moxfield primers are Markdown plus a handful of Moxfield-only constructs, minus
raw HTML (stripped, to stop people wrecking the site's design). Get the syntax
wrong and it does not error - it renders as literal text in front of everyone
reading the deck. So: compose here, then always check **Preview** in the editor.

## Contents

- [The one that bites everyone: mana symbols](#the-one-that-bites-everyone-mana-symbols)
- [Card links](#card-links)
- [Standard Markdown that works](#standard-markdown-that-works)
- [Moxfield-only constructs](#moxfield-only-constructs)
- [Collapsible panels and the table of contents](#collapsible-panels-and-the-table-of-contents)
- [Structuring a primer](#structuring-a-primer)

## The one that bites everyone: mana symbols

**`{B}{B}{B}` does not render on Moxfield.** It has been verified in the live
editor: curly-brace notation - the Scryfall/Oracle convention, and what most
primer templates tell you to write - comes out as the literal characters
`{B}{B}{B}` in the published primer.

Moxfield's syntax is `[[symbol:ABBREVIATION]]`:

```
[[symbol:1]][[symbol:b]][[symbol:b]]     a 1BB cost
[[symbol:t]]                             wrong - use tap
[[symbol:tap]]                           the tap symbol
```

Abbreviations (lower case):

| Kind | Abbreviations |
| --- | --- |
| Colours | `w` `u` `b` `r` `g` |
| Generic | `0` `1` `2` ... `20`, `x` |
| Colourless / snow / energy | `c` `s` `e` |
| Phyrexian | `p`, and `wp` `up` `bp` `rp` `gp` |
| Two-brid | `2w` `2u` `2b` `2r` `2g` |
| Hybrid | `wu` `wb` `rw` `gw` `ub` `ur` `gu` `br` `bg` `rg` |
| Symbols | `tap` `untap` `loyalty` `saga` `paw` |

If you need one that is not listed, the editor's symbol button on the toolbar
inserts the correct code - that picker is authoritative, this table is a
convenience.

A note on taste: symbols are for costs and requirements where the pips are the
point ("the `[[symbol:b]][[symbol:b]][[symbol:b]]` requirement on turn three"). A
primer peppered with symbols in ordinary prose reads worse, not better.

## Card links

Wrap the card name in double brackets. Hovering shows the card image; clicking
goes to the card page.

```
[[Ensoul Artifact]]      renders as a hover link
[[fatal push]]           case-insensitive - this works
[[This Town Aint Big Enough]]   broken - punctuation must match exactly
```

Punctuation matters even though case does not: `[[This Town Ain't Big Enough]]`,
`[[Urborg, Tomb of Yawgmoth]]`, `[[Otawara, Soaring City]]`. A near-miss renders
as plain bracketed text and looks careless, so check the exact names against the
decklist you read from the deck page.

A set-specific printing can be pinned as well; if you need that, insert it with
the editor's card button rather than guessing the separator, since plain
`[[Card Name]]` is what a primer wants in almost every case.

Link the card on **every** mention, not just the first. Readers scan primers; a
hover on the third mention is as useful as on the first, and Moxfield's audience
expects it.

## Standard Markdown that works

- **Headers** `#` through `######`. Put a blank line above *and* below - Moxfield
  is strict about this and a header jammed against text will not render.
- **Bold** `**text**`, **italic** `_text_`, **strikethrough** `~~text~~`.
- **Unordered lists** start a line with `*`. **Ordered lists** with `1.`.
  Nest with **4 spaces** - not tabs, which break the list. Leaving a **blank line
  between items** makes the list *loose*: each item renders in its own paragraph
  with real spacing. Tight lists read as a wall; loose lists are worth the extra
  blank lines in anything a reader has to work through.
- **Links** `[text](url)`, with an optional hover title:
  `[Moxfield](https://moxfield.com "a deck building site")`.
- **Images** `![alt](url)`. They cannot be resized in Moxfield - resize the source
  image if it matters. An image can be a link: `[![alt](imgurl)](linkurl)`.

**Card links do not work inside code blocks.** Anything in a fenced block or
backticks renders literally and in monospace, so `[[Card Name]]` there is dead text
rather than a hover. Reserve code blocks for things that genuinely are code or
fixed-width data, and never for a list of cards.

Raw HTML is removed. Tables are not part of the documented dialect - if you want a
grid (a sideboard guide, say), a definition-style list reads fine and is safe.

## Moxfield-only constructs

| Construct | Syntax |
| --- | --- |
| Card | `[[Card Name]]` |
| Mana / other symbol | `[[symbol:b]]` |
| User mention | `[[user:username]]` |
| YouTube video | `[[youtube:VIDEO_ID]]` |
| YouTube playlist | `[[playlist:PLAYLIST_ID]]` |
| Centred block | `===center` / `===endcenter` on their own lines |
| Spoiler (click to reveal) | `\|\|hidden text\|\|` - two vertical bars either side |

## Collapsible panels and the table of contents

Moxfield builds the primer's table of contents from **collapsible panel titles**,
not from headers. That is the one structural decision worth making deliberately:
panels give the reader navigation and let them skip to the section they want.

```
===accordion
===panel: Gameplan
Content goes here, including [[Card Links]] and lists.
===endpanel
===panel: Sideboard Guide
More content.
===endpanel
===endaccordion
```

Add `|autocollapse` to start every panel closed - good for a long primer where
the reader should choose, bad for a short one where collapsing everything just
hides the whole document:

```
===accordion|autocollapse
```

Rule of thumb: **four sections or fewer, use headers**; a long primer with
optional sections (e.g., sideboard guide, matchups, changelog) earns panels, because
then the table of contents is doing real work.

## Structuring a primer

Short primer - headers only:

```
### Is This Deck For You?
...

### Gameplan
...
```

Long primer - panels, so the reader gets a table of contents:

```
# Deck Name

_One-line hook._

===accordion
===panel: Is This Deck For You?
...
===endpanel
===panel: Gameplan
...
===endpanel
===endaccordion
```
