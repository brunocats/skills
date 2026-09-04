# Operating Moxfield through the browser

Everything here is about *mechanics*: which page, which control, what to type,
how to check it worked. Nothing here is about what makes a good name or a good
primer - that lives in the craft references.

## Contents

- [Why the browser](#why-the-browser)
- [Getting a browser](#getting-a-browser)
- [Working with a heavy single-page app](#working-with-a-heavy-single-page-app)
- [URL map](#url-map)
- [Publishing cheaply](#publishing-cheaply)
- [Reading the current state of a deck](#reading-the-current-state-of-a-deck)
- [Name, description, visibility, format](#name-description-visibility-format)
- [Main deck and sideboard](#main-deck-and-sideboard)
- [Hubs](#hubs)
- [Deck image](#deck-image)
- [Folders](#folders)
- [Primer](#primer)
- [Creating a new deck](#creating-a-new-deck)
- [Verification checklist](#verification-checklist)

## Why the browser

Moxfield's API is private - their FAQ says so, and access is by arrangement only,
via `support@moxfield.com`. There is no key the user can hand you, and no supported
way to PUT a deck. Do not reverse-engineer `api2.moxfield.com` from the page: it
breaks without warning, and it is not what the user agreed to.

Driving the logged-in browser also has a real advantage - it goes through the same
validation a person gets, so an illegal card or a malformed list surfaces as a
visible error instead of silent corruption.

## Getting a browser

The user's Chrome, via the Claude in Chrome tools (`mcp__claude-in-chrome__*`),
is the default - that is where they are signed in to Moxfield. If those tools are
deferred, load them in **one** call:

```
ToolSearch: select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__read_page,mcp__claude-in-chrome__get_page_text,mcp__claude-in-chrome__find,mcp__claude-in-chrome__form_input,mcp__claude-in-chrome__javascript_tool,mcp__claude-in-chrome__tabs_create_mcp,mcp__claude-in-chrome__tabs_close_mcp
```

Call `tabs_context_mcp` once before anything else, and create your own tab rather
than hijacking one of theirs.

If the extension is not connected, say so plainly and ask whether to use the
built-in Claude browser instead (`mcp__remote-devices__Claude_Browser__*`, same
verbs). The built-in browser has its own Moxfield session, so the user may need
to sign in there once. Do not switch browsers on your own initiative - one of
them is the user's real Chrome, and that difference matters to them.

If the deck page shows a signed-out header or no `Primer / Playtest / Edit / Buy /
More` toolbar, the session is not logged in as the deck's owner. Stop and say so
rather than clicking around.

## Working with a heavy single-page app

Moxfield is a large React app that routes client-side. Practical consequences:

- After `navigate`, the first read often returns `Loading Moxfield...` or
  `Moxfield is taking longer than expected...`. That is normal. Wait a few seconds
  and read again before concluding anything is wrong.
- The document title and `location.href` update before the content does. Trust the
  rendered page, not the URL.
- Prefer `read_page` / `find` and click by `ref`, over screenshot coordinates.
  Refs survive layout shifts and ad slots moving things around; coordinates do not.
  Use a screenshot when you need to see state a ref cannot express (which radio is
  selected, whether a modal is really open) - and see *Publishing cheaply* for why
  those should stay rare.
- Ad banners inject and reflow the page. Re-read before clicking if more than a
  few seconds have passed.

## URL map

`<id>` is the deck id from the URL, e.g. `https://moxfield.com/decks/PUygARfuB3-lLY78f7_OQA`.

| Page | URL | What is there |
| --- | --- | --- |
| Deck | `/decks/<id>` | Name, format chip, hub chips (each with an `x`, plus a `+` chip), "Change deck description", "Change card image", counts, toolbar: `Primer` `Playtest` `Edit` `Buy` `More` |
| Bulk edit | `/decks/<id>/edit` | One text box, showing whichever board is selected. Board tabs: Main Deck, Sideboard, Considering, Other. Buttons: Swap board with, Import from File, Cancel, Save, Save & Continue Editing |
| Primer editor | `/decks/<id>/primer` | Markdown editor with toolbar, `Preview` / `Fullscreen` / `+ Primer Badge`, then Cancel / Save Primer |
| Settings | `/decks/<id>/settings` | Name, Description, Visibility, Format, Allow Comments, Allow Primer to be Duplicated, Convert to Package, Cancel / Save Settings |
| History | `/decks/<id>/history` | Version history - the user's undo, and your evidence a save landed |
| Your decks | `/decks/personal` | The user's decks and folders, `+ New Deck` / `+ New Folder` |

The `More` menu on the deck page holds: Import, Export, Duplicate, Compare, View
History, Get Playtest Cards, Get Deck Registration, sell/rent links, Add to
Collection, Add to Wish List, Change Authors, Update to Cheapest, Update to
Preferred, **Settings**, Delete. Navigating straight to `/decks/<id>/settings` is
faster than opening the menu.

**Only `Export` and `Settings` are in scope.** Everything else in this menu either
destroys the deck, rewrites every printing in it, or writes somewhere outside it -
Duplicate, Change Authors, Update to Cheapest, Update to Preferred, Delete, Import,
Add to Collection, Add to Wish List. The same goes for `Convert to Package` on the
settings page and `Swap board with` / `Import from File` on the bulk-edit page. If
the user's phrasing seems to point at one, ask.

## Publishing cheaply

A publish that takes thirty browser calls costs the user real money, because in a
long conversation every call re-sends the accumulated context. Cost tracks the
**number of calls**, not the size of any one of them. A careless publish and a
careful one differ by roughly five times.

Four rules, in order of how much they save.

**Navigate once.** Moxfield is a single-page app, so its own links route without a
page load. Open the deck page once with the browser tool, then move between the
deck, its settings and its primer by clicking in-app links from inside your
JavaScript. One navigation for the whole session instead of one per operation - and
crucially, the page context survives, so anything you parked on `window` is still
there.

**One call per operation, not per step.** Wait for the app, act, verify, and return
the verification - all in the same call. A separate call to check what you just did
doubles the cost of every operation for information the acting call could have
returned.

**Park the long text on `window`.** Send a primer's text across once, into
`window.__P`, then reuse it for the paste, for a retry, and for the length check.
Re-sending seven thousand characters because you navigated away is a self-inflicted
cost - and it is why the previous rule matters.

**Read the DOM, not screenshots.** A screenshot is expensive and tells you less
than three lines of `querySelector`. Take one only when the visual layout is the
question.

### The shape of a cheap operation

```js
// Everything in one call: route, wait, act, verify, report.
async function ready(sel, tries = 15) {
  for (let i = 0; i < tries; i++) {
    const el = document.querySelector(sel);
    if (el && !/Loading Moxfield/.test(document.body.innerText)) return el;
    await new Promise(r => setTimeout(r, 2000));
  }
  return null;
}

// route in-app rather than navigating
document.querySelector('a[href$="/primer"]').click();
const ed = await ready('[contenteditable=true]');
if (!ed) return JSON.stringify({ error: 'editor never appeared' });

// ... act ...

// return the verification, do not spend another call on it
JSON.stringify({ len: ed.innerText.length, expected: window.__P.length });
```

The code above is illustration, not a bundled script: it lives in a markdown fence,
it enters the assistant's context before it runs, and it is regenerated for the job
at hand. This skill ships no executable files at all, which is a deliberate security
property rather than an oversight - a script whose source never reaches the
assistant's context is exactly the attack the published Agent Skill threat models
warn about. Keep it that way: if an operation needs code, write it in a fence.

### A full publish, budgeted

| Call | What it does |
| --- | --- |
| 1 | Open the deck page |
| 2 | Read current state: name, hubs, description, and the list via More > Export |
| 3 | Settings: route, set description and visibility, save, verify |
| 4 | Image and hubs: open each modal, set, save, verify |
| 5 | Park the primer text on `window` |
| 6 | Primer: route, select all, paste, verify length |
| 7 | Preview, check the rendering, save, verify |

Seven calls, not thirty. Steps 3 and 4 combine when both are small.

## Reading the current state of a deck

Read lazily. Each page load is slow, so open only what the task needs - and nothing
at all for a decklist the user has already given you.

**The whole list, in one place: More > Export.** From the deck page, the `More`
menu's `Export` item opens an Export Options panel whose Deck List text box already
contains the full list - main deck, then a `SIDEBOARD:` section - one card per line
with quantities, set codes and collector numbers. Read that text box's value
directly from the DOM. Do **not** click `Copy Plain Text` to get at it: the text is
already readable, and clicking would overwrite whatever the user has on their
clipboard. This is the best read available - no extra page load, and it does not
care how the user has configured their deck view.

**Export is the read for everything except a list change.** A list change reads the
bulk-edit box of the board it is about to write and builds the new text from that
box's exact contents - never from an Export block, which concatenates boards and
would inject a literal `SIDEBOARD:` line and the whole sideboard into your main deck.

Export covers **Main Deck and Sideboard only**. When Considering or Other matters -
notably for the image picker, which offers cards from every board - read those
boards on `/decks/<id>/edit`.

**The rest of the deck page.** The header gives name, format, hubs, description and
the main/sideboard counts. The card list is fully rendered too, but quantities live
in editable `<input>` fields on the owner's view, so scraping `innerText` gives you
card names with no numbers - a quiet way to build a wrong list. Read input values
alongside names, or just use Export.

**Other pages, only when the task needs them:**

- `/decks/<id>/edit` - one text box holding the selected board, in the format you
  write back. Click the board's tab, read it, then **Cancel** out. Required
  immediately before any list change, always.
- `/decks/<id>/primer` - for the owner this opens the editor, so it gives the
  primer's raw source rather than rendered output. Read before revising; Cancel out.
- `/decks/<id>/settings` - visibility, exact description, format. Read once before
  the first push of a session: the ledger has to name the deck's current visibility,
  and you cannot state it without looking.

**Finding a deck the user named but did not link:** the search box on
`/decks/personal` filters their decks by name - faster and less error-prone than
paging the list. Confirm the match by name and URL before working on it.

## Name, description, visibility, format

All four live on `/decks/<id>/settings`.

- **Name** - free text.
- **Description** - max 140 characters, plain text. This is *not* the primer. It is
  the one-line blurb under the deck title, and it appears nowhere else on the site.
  What goes in it is covered in `references/deck-naming.md`.
- **Visibility** - three radio buttons:
  - **Public** - listed, searchable, appears in Moxfield's explore pages
  - **Unlisted** - reachable by anyone with the link, not listed or searchable
  - **Private** - only the owner
- **Format** - a long dropdown (Standard, Pioneer, Modern, Legacy, Vintage, Pauper,
  Commander / EDH, Duel Commander, Brawl, Historic, Timeless, Alchemy, Oathbreaker,
  Premodern, Old School, the Highlander variants, ... and `None`). It drives the
  legality filter in card search, so changing it can make cards in the deck show
  as illegal. Only change it when the user asks.

Set the fields, click **Save Settings**, then re-read the deck page to confirm.

Visibility is the setting the confirmation ledger must always name - see step 3 of
the push in SKILL.md.

## Main deck and sideboard

Use `/decks/<id>/edit` (Bulk Edit). Each board is a plain text box, one card per
line:

```
AMOUNT CARDNAME (SETCODE) COLLECTORNUMBER *F*
```

Only `AMOUNT CARDNAME` is required; set code, collector number and `*F*` (foil) are
optional. Examples:

```
4 Ensoul Artifact
1 Counterspell (CMR) 632 *F*
3 Fatal Push
```

Lines may also carry `#tags`. Ignore them - Moxfield restores the user's card tags
after a bulk edit, so they are not yours to manage.

Boards: **Main Deck**, **Sideboard**, **Considering**, **Other** - tabs over a
single text box, so only the selected board is on the page at any moment. Commander,
companion and similar zones live under the appropriate board for the format - read
what is there rather than assuming.

The things that go wrong:

- **The box is the whole board.** Saving replaces it. Build the new text from the
  list you just read, apply the user's changes to that, and write the complete
  board back. Never type "just the changed lines".
- **Card names must be real.** A typo becomes a failed line on save. If the user
  gives you a name you are not sure of, check it (Scryfall) before staging it.
  Never invent a printing.
- **Non-English cards cannot be added through bulk edit** at all.
- **Getting the text in.** Long replacements have their own failure modes, and
  picking the wrong board's box overwrites it. `references/text-injection.md`.
- **Save vs Save & Continue Editing.** `Save` returns to the deck page, which is
  where you want to be to verify counts. Use `Save & Continue Editing` only when
  you have another board to edit in the same pass.

After saving, check the counts in the deck page footer (`60 main deck / 15
sideboard`) against what you intended. A wrong count is the fastest signal that a
line failed to parse.

## Hubs

Hubs are Moxfield's public categorisation tags - the chips under the deck name
(`AGGRO`, `ARTIFACTS`, `DIMIR (UB)`, `TEMPO`). They are set from the deck page:
click the `+` chip next to the existing hubs. The modal is titled **Select
Themes**: a search box and a long checkbox list, then **Save**.

- Check to add, uncheck to remove, then Save once. You can also remove a hub
  directly with the `x` on its chip.
- The modal's list is the authority on which hubs exist; `references/hubs.md`
  covers which ones to pick.
- `Primer` is a hub like any other, and the primer editor's `+ Primer Badge` button
  is just a shortcut that adds it. Do not click it on your own initiative - it is a
  public hub change, so it belongs in the ledger and goes through the gate like every
  other hub. **But do put it in the ledger** whenever you are publishing a primer
  with real content: it is what makes the deck findable by people browsing for decks
  that have one, and forgetting it wastes the work.

Which hubs to pick is in `references/hubs.md`.

## Deck image

The picture on the deck's banner and deckbox. Set it from the deck page: click
**Change card image** under the deck header. The modal is titled **Change Main
Image** and holds an image preview, a single **Selected Card** dropdown, and
Cancel / Save.

The constraint that matters: **the dropdown only lists cards already in the deck.**
Every board counts - main deck, sideboard, considering - and the deck's tokens are
in the list too. There is no card search and no arbitrary image upload. So when the
user names a card:

- **In the deck** - select it and Save.
- **Not in the deck** - say so rather than picking something close. The options are
  to add the card first (which changes the list, a separate decision) or to choose
  a different card. Do not quietly substitute.

Each option is a specific printing, shown as `Card Name (SET) #number` - the
printing the deck actually uses. Picking a different printing's art therefore means
changing the printing in the decklist first. Double-faced cards appear as both
faces (`Clearwater Pathway` and `Murkwater Pathway` are separate options), so a
user asking for one of those has a real choice between the two arts.

The dropdown is a plain `<select>`, so setting it by value and dispatching a
`change` event works if clicking is awkward - but read the preview back before
saving, since the value is a Moxfield card id rather than a name.

Choosing well: the image is the deck's face in every list it appears in. The
payoff card the deck is named after is the usual right answer - the same card the
name anchors on, so the box and the title agree.

## Folders

Folders are the user's private filing on `/decks/personal` (e.g., "Pioneer",
"Commander", "Proxies", etc.) - freely named, invisible to everyone else, and
unrelated to hubs. `+ New Folder` creates one, but only when the user has asked for
that folder by name. A deck is moved from the per-row menu on that page; the
wording of that menu changes, so read it and act on what is there rather than
clicking a remembered position.

A folder move is private rather than public, but it is still a write to the user's
account: it goes in the ledger and through the same gate as everything else, and
never touches a deck that was not part of the request.

## Primer

`/decks/<id>/primer` opens the editor for the owner. Layout: `Preview`,
`Fullscreen`, `+ Primer Badge` above a toolbar (header, bold, italic, strike,
centre, spoiler, lists, link, image, YouTube, panel, accordion, card, user,
symbol), then the editor body, then **Cancel** / **Save Primer**.

Workflow that avoids pain:

1. Read the existing source first (the editor shows raw source, not rendered).
2. Compose the new primer in the chat and get it approved there.
3. Put the text in - see `references/text-injection.md`, which also covers why a
   save can look successful and not be - then click **Preview** and read the
   rendered result. `Preview` toggles in place and becomes `Back to Edit`. Preview is the only reliable check that your syntax works -
   card links, symbols, accordions and centring all fail *silently* as literal
   text if the syntax is off.
4. Click **Back to Edit**, then **Save Primer**.

`Cancel` discards the buffer, so an unsaved experiment costs nothing. Nothing is
saved until **Save Primer** is clicked - which also means a session that ends with
an un-clicked Save Primer changed nothing.

Moxfield's primer dialect and its traps are in `references/moxfield-markdown.md`.
Read it before writing a primer - plain Markdown mana notation like `{B}{B}` does
**not** render on Moxfield.

## Creating a new deck

`+ New Deck` at the top right of any page (the `+` on narrow screens) opens the
Add Deck modal:

- **Name** - can be changed later, so a working name is fine.
- **Format** - and, for Commander / Oathbreaker / Brawl, the commander or signature
  spell. `None` at the bottom of the list for a plain card list.
- **Visibility** - Public / Unlisted / Private. Default to **Private** unless the
  user said otherwise; a deck can always be published later, but an accidental
  public deck has already been seen.
- **Starting list** - optional: paste a list (same `AMOUNT CARDNAME ...` format as
  bulk edit), upload a `.txt`, or import from a TappedOut or Archidekt URL.

After creation, capture the new deck id from the URL and continue with the normal
operations above.

## Verification checklist

After a push, confirm from the rendered pages rather than from the fact that you
clicked Save:

- Deck page shows the new name and description.
- Footer counts match the intended main deck / sideboard sizes.
- Hub chips match the agreed set.
- The banner image is the card that was agreed.
- The primer renders: card links are links, mana symbols are symbols, accordion
  panels are collapsible, no stray `===panel` text.
- Visibility badge (or its absence) matches what was agreed.
- `/decks/<id>/history` shows a new version - proof the save reached the server.

Then report to the user what is live, with the deck URL.
