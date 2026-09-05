# Putting text into Moxfield's editors

The bulk-edit boxes and the primer editor are custom React components, and this is
where a save goes wrong silently. Two different jobs, two different techniques.

## Small, surgical edits - prefer these

Fixing a phrase, correcting syntax, changing a number. Select exactly the text to
replace and let the editor handle the input, so the rest of the document is never
retyped and cannot be mangled. Verified to work in the primer editor:

```js
const ed = document.querySelector('[contenteditable=true]');
ed.focus();
const walk = document.createTreeWalker(ed, NodeFilter.SHOW_TEXT);
for (let n; (n = walk.nextNode()); ) {
  const i = n.nodeValue.indexOf(TARGET);
  if (i < 0) continue;
  const r = document.createRange();
  r.setStart(n, i); r.setEnd(n, i + TARGET.length);
  const sel = window.getSelection();
  sel.removeAllRanges(); sel.addRange(r);
  document.execCommand('insertText', false, REPLACEMENT);
  break;
}
```

Run it once per occurrence and check the length delta each time - it should change
by exactly `REPLACEMENT.length - TARGET.length`.

## Replacing a whole board or a whole primer

**First, make sure you are on the right board.** The bulk-edit page has one text
box and the board tabs swap which board it holds, so the box you grab is whichever
tab is selected - not whichever index you ask for. Saving replaces that whole
board, so writing with the wrong tab active overwrites it.

Click the tab you intend to edit, then confirm before writing: the box's accessible
name carries the board and its count (`Main Deck (60)`, `Sideboard (15)`), and its
current first line should match the board you just read. If neither check is
available, do not write - re-read the board and try again.

### The primer editor: dispatch a paste event

For a whole-document write into the `contenteditable` primer editor, **the
`execCommand` route above does not work** - and it fails silently, which is worse
than failing loudly. `insertText` over a whole-document range writes into the DOM
but never reaches the editor's internal model: the text is visible in the element,
`innerText` reports it correctly, and then Preview renders nothing and Save saves
nothing. Verified in the live editor.

The surgical technique works because it selects text that already exists in the
model. An empty editor has no such text, so there is nothing to anchor to.

What works is a synthetic paste, which the editor handles through its own paste
path. **Replacing** existing content needs one more step: a JS `Range` selection is
invisible to the editor's model in the same way, so a paste on top of one *appends*
rather than replaces - you end up with both versions and a save that looks fine.
Select all with a real key event first:

1. `document.querySelector('[contenteditable=true]').focus()`
2. Send `cmd+a` (or `ctrl+a`) as an actual keypress through the browser tool, not
   through JS. Confirm it took: `window.getSelection().toString().length` should be
   the length of the existing document.
3. Dispatch the paste:

   ```js
   const ed = document.querySelector('[contenteditable=true]');
   const dt = new DataTransfer();
   dt.setData('text/plain', text);
   ed.dispatchEvent(new ClipboardEvent('paste', {
     clipboardData: dt, bubbles: true, cancelable: true,
   }));
   ```

Into an empty editor, step 2 is unnecessary and the paste alone is enough.

Hold the text on `window` (e.g., `window.__P`) rather than re-sending it for the
paste, the retry and the length check - `moxfield-operations.md` §Publishing cheaply
explains why that matters and how to keep the page context alive.

This is not the same as the clipboard: nothing is written to the user's clipboard
(`navigator.clipboard.writeText` is refused in the automation context anyway), and
no keystroke is simulated. The event carries its own data.

Check the length afterwards, and read what it tells you:

- **Source length (within a character or two)** - clean.
- **Old length plus new length** - the selection did not register and you have
  appended. Select all again with a real keypress and repeat.
- **Noticeably longer, with runs of blank lines** - the text is in the DOM but not
  the model. Preview will render nothing and the save will be empty.

### Other fields

For a real `<textarea>` or `<input>` (the bulk-edit boxes), React ignores a plain
`el.value = ...`, so use the native setter and dispatch an input event:

```js
const setter = Object.getOwnPropertyDescriptor(
  window.HTMLTextAreaElement.prototype, 'value').set;
setter.call(el, newText);
el.dispatchEvent(new Event('input', { bubbles: true }));
```

For a `<select>` (the deck image picker) the same trick works with
`HTMLSelectElement` and a `change` event.

## Always read back before saving

For a primer, click **Preview** and confirm the rendering, not just the text:
card links become links, mana symbols become symbols, and no `[[` or `===`
survives. An empty preview means the text never reached the editor's model. For
a board, count the lines and check the first and last. Silent truncation and a
mangled first line are the common failures, and both are invisible until someone
opens the deck.

**Saving the primer gives no visible confirmation** - the editor stays open and
looks unchanged whether or not the save landed. The only reliable check is to
reload `/decks/<id>/primer` and read the source back: an unsaved buffer disappears,
a saved one comes back with your text. Do that before telling the user it is done.
