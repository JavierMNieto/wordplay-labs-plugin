{{!--
================================================================================
THE .WPLAY FORMAT: THE ONE PLACE IT IS DEFINED

Labs serves this file at /format and, for an agent, at /format.md
(apps/wordplay/src/lib/format-reference.ts); the builder's guide includes it
whole (the spec binding in packages/player/prompts/lesson-guide.md). The
parser in packages/format/src implements it, and spec.test.ts fails when
they drift: every ```html block below must parse with no problems, and each
attribute table must equal the parser's set. Edit the prose here; the
numbers in double braces are bound from packages/format/src/limits.ts.

As agreed on 4 and 5 October 2026, from the discussion posted to the
Wordplay Labs group:
https://wordplaytalk.com/p/cmuun53g200122ppbn1eb314m, simplified on
5 October, and on 8 October, after the company meeting of 7 October, to
data-expect and data-context, with document.wordplay for scripts and sections
as the steps that count; and on 9 October with data-tutor, the one place
the tutor writes, and document.wordplay.say. The decisions behind it, and its
history, are in docs/wordplay-format.md.

Licensed under Apache-2.0 with the rest of packages/ (packages/LICENSE).
--}}
# The `.wplay` format

## What a wordplay is

A wordplay is a teacher's understanding, frozen into a file, that a tutor
defrosts in conversation with one student at a time. The file holds three
things: what the student plays with, when each part of it appears, and what
the teacher prepared for the tutor. A tutor takes those three and holds the
session the teacher would have held.

## The file

A `.wplay` file is an HTML document with four attributes on it. ARIA added
attributes to HTML so a screen reader could read a page; these four add
what a tutor needs on top of that.

| Attribute | On any element, means |
|---|---|
| `data-wordplay="1"` | On `<body>`: this document is a wordplay. |
| `data-context` | The student never sees this element. The tutor reads it, and operates it if it is a control. |
| `data-expect="…"` | This element is given once a condition holds, judged by the tutor from the conversation, or said by the page's script. |
| `data-tutor` | On an `<output>`: the tutor writes it, by its accessible name. What it writes replaces what was there. |

Everything else is HTML, written to ARIA, and is shown as HTML. A relative
URL in it resolves against where the file is. What is logical, checking an
answer or counting a game, is the teacher's script (Scripts, below).

## Rules

**One rule.** An element with a condition is given once its condition
holds: to the student, or, if it is marked `data-context`, to the tutor.
Until then it is not given at all, with everything it holds. An element
without a condition is given with its parent. The body is always shown.

**Steps and reveals.** A `<section>` the student sees that carries a
condition is a step, and steps are the progress (below). Any other element
the student sees with one is a reveal, a hint or a figure, given the same
way and not counted; a hint is `<aside data-expect="…">`. A condition goes
on what the student should see next, written as the moment that earns it.

**Order.** A condition is judged once its element's parent has been given.
Conditions do not wait for one another otherwise, so a hint the student
never needs blocks nothing after it. Two steps that must come one after the
other nest, or say in their conditions what they follow.

**Context.** A context element holds whatever the teacher wants the tutor
to read; the tutor is given its text, with inline markup kept as written,
so math and code survive. Context with no condition is given as soon as its
parent is; with one, once the condition holds. Context nests by nesting.

**The condition.** `data-expect` is prose, the moment as the teacher would
recognize it, judged by the tutor or said by a script (`document.wordplay.met`).

**Ids.** Each step, reveal and context element is named by its own `id`,
else numbered in document order: `s1`, `s2`, … for steps, `r1`, … for
reveals, `c1`, … for context. The tutor and scripts say what holds by id.

**Reading the page.** The tutor is given the page as a screen reader would
read it: the accessibility tree (role, name, state) of what is shown, and
what its live regions have announced. Nothing about how the page is drawn
reaches it. A page that works for a blind student works for the tutor.

**Acting on the page.** Anything a student could do with a keyboard, the
tutor can do by accessible name: press a button, set a field, choose an
option. An element marked `data-context` is removed from the student's page
and their screen reader, and placed in the tutor's tree alone.

**Writing on the page.** An `<output>` marked `data-tutor` is the tutor's to
write, by its accessible name: what it writes is the element's text,
replacing what was there. Nothing else on the page takes the tutor's words,
and how they are drawn is the teacher's.

**Privacy.** Every element marked `data-context` is stripped before anything
reaches the student's browser. This is the one rule a host must never break.
A step or reveal reaches the browser only once given, without its
`data-expect`, which is often the answer. Where a host lets readers share
(Wordplay's `share`, below), what a reader shares is seen by every reader
of the wordplay, with their profile only where the teacher asks for names
and the reader has not chosen to be anonymous.

**Unknowns.** A host ignores an attribute it does not know and shows the
element as HTML.

**What the tutor is given at any moment.** The tree of what is shown, its
announcements, what the page told it since its last turn, and what was
worked out on a calculator. The context
given so far. Each part not yet given whose parent has been, by id and
condition, so it knows what to watch for. The steps the student skipped.
Where a student is is never in the file; it is worked out from their
conversation. With no conversation, a host shows the whole wordplay.

## Scripts

The teacher's scripts run as on any page. Before the body is read, the
host puts one object on the document, `document.wordplay`, frozen, with
these members:

| Member | Does | At most |
|---|---|---|
| `met(id)` | That element's condition holds: the host gives it, as if the tutor had said so. An id not waiting is ignored. | |
| `tell(text)` | The text reaches the tutor at its next turn. A live region is read out to the student too; this is not. | {{maxAnnouncement}} characters |
| `say(text, chat?)` | A message from the student, as if typed, in the chat named or the tutor's own: the tutor answers it now, and is told it came from the page. Returns a promise of the reply as the student reads it, once it ends; refused, it rejects with why. `tell` waits for the next message; `say` is one. | {{maxSaid}} characters, {{maxPageSays}} a minute |
| `keep(value)` | Keeps one JSON value, the page's own state, with the conversation; each call replaces the last. A larger value is dropped, with a warning in the console. | {{maxKept}} bytes |
| `kept` | The value last kept in this conversation, or `undefined`; there before the first script runs. | |

**`wordplay:met`** is fired on a part as it is given, bubbling, its id in
`event.detail.id`, and again for each part already given when the page is
drawn, in document order, so a script knows what is on the page whether
it was given now or before.

**The page is the conversation's.** The host keeps each control's value,
typed, checked or chosen, by its id, else its name, else its accessible
name, and which page was open; drawing the page again, it sets them back
and fires `input` and `change`, so a script that draws from its fields
draws again. What lives only in a script's variables is `keep`'s:

```js
// A pile drawn in <output id="pile">, taken from by <button id="take">.
const game = document.wordplay?.kept ?? { pile: 21 };
const draw = () => (document.querySelector("#pile").textContent = game.pile);
document.querySelector("#take").onclick = () => {
  game.pile -= 1;
  document.wordplay?.keep(game);
  draw();
  if (game.pile === 0) document.wordplay?.met("emptied");
};
draw();
```

The kept value is never the tutor's, which reads the page as shown. It is
one conversation's: starting over starts from the file. It is one value,
not a store: what the page needs to draw itself again.

Opened as plain HTML a page has no `document.wordplay`, so a script calls
`document.wordplay?.met("solved")`. A host may add members of its own
("What Wordplay adds"), which a script tests for:
`document.wordplay?.share?.(…)`. A script runs in the student's browser,
which can read it: a secret goes in `data-context` or behind a
`data-expect`, never in a script.

## Progress

**Progress** is the steps given over the steps in the file, nested ones
counted flat; it says how many, never in what order, since steps do not
wait for one another. **Done** is every step given; a wordplay with no step
is never done. **Skip**, where a host offers it, gives one waiting step as
if its condition held, and nothing else; what is inside it then waits as
usual. A step with `data-skip="false"` has none, and the student must earn
it. A step skipped counts, and the tutor is told of it. A part `met` gives
is the browser's report, counted apart from what the tutor gave.

## Example

Page 2 of *The game from Marienbad*:

```html
<!doctype html>
<html lang="en">
<body data-wordplay="1" data-chat data-progress data-skip>

<div data-context>The game is Nim with one row. The computer plays perfectly.</div>

<section aria-label="One row, small steps">
  <p>There is only one row of 21 matches. On your turn you take 1, 2 or 3. The
  player who takes the last match wins. You go first. Play a few games, look
  for a pattern, and tell your tutor your idea.</p>

  <div class="matches" role="group" aria-label="21 matches. Take 1, 2 or 3.">…</div>
  <p aria-live="polite">Your move.</p>
  <button>Start again</button>
  <form aria-label="Set up a position" data-context>…</form>

  <aside id="stuck" data-expect="they have lost three games in a row">
    <p>Try a smaller game first: who wins with 4 matches on your turn?</p>
  </aside>
  <script>
    // The game calls gameOver at the end of each game.
    let lost = 0;
    function gameOver(won) {
      lost = won ? 0 : lost + 1;
      if (lost === 3) document.wordplay?.met("stuck");
      if (won) document.wordplay?.tell("They beat the computer from 21 matches.");
    }
  </script>

  <div data-context data-expect="they say the trick is to always take 1">
    It beats a careless player and loses to the computer. Ask what happens from 4.
  </div>
  <div data-context data-expect="they ask why 4 is the bad number">
    From 4, any move leaves 1, 2 or 3, and the opponent takes the rest.
    <div data-context data-expect="they ask whether it works when the last stick loses">
      Everything shifts by one: 1, 5, 9, 13, 17, 21.
    </div>
  </div>

  <section aria-label="One row solved"
    data-expect="they say, in their own words, always leave a multiple of 4">
    <p>You found the winning strategy.</p>

    <p><strong>Exercise.</strong> On <em>Fort Boyard</em>, candidates play this
    game against the Master of Time. 21 sticks, take 1, 2 or 3, the candidate
    goes first, and the player who takes the last stick <strong>loses</strong>.
    What is the strategy now? Can the candidate win at all?</p>

    <div class="matches" role="group" aria-label="21 sticks. Take 1, 2 or 3. The last stick loses.">…</div>
    <p aria-live="polite">Your move.</p>

    <section aria-label="Fort Boyard solved" data-skip="false"
      data-expect="they say the candidate cannot win from 21">
      <p>Right: 21 is one more than a multiple of 4, so the Master of Time
      always wins.</p>
    </section>
  </section>
</section>

</body>
</html>
```

Opened as HTML, it is the wordplay without the tutor. As a tree, conditions
on the edges:

```
context c1  (always)
context c2  (always: the set-up form)
reveal stuck  ── expect: they have lost three games in a row
context c3  ── expect: they say the trick is to always take 1
context c4  ── expect: they ask why 4 is the bad number
└── context c5  ── expect: they ask whether it works when the last stick loses
step "One row solved"  ── expect: they say, in their own words, always leave a multiple of 4
└── step "Fort Boyard solved"  ── expect: they say the candidate cannot win from 21
```

Two steps: the hint counts for nothing, and the second has no Skip.

## What Wordplay adds

The above is the standard. Wordplay's site adds tooling that another host may
ignore, and the file loses nothing. Nothing of it is drawn unless the file
asks: a wordplay with none of it is its HTML, in the site's reading column.
What is drawn around the page is declared on the `<body>`, each a boolean
attribute with no value, as `<video controls>` declares its player; what is
drawn in the page is an element in it.

- The `<body>`'s declarations: `data-chat`, the tutor's chat beside the
  page; `data-progress`, progress as a thin bar over the page that only
  shows it; `data-pages`, each `<section>` directly in the body without a
  condition drawn as a page, with the pages along the foot; `data-skip`,
  Skip at each waiting step, where it will appear.
- With `data-pages`, a link to an id on another page turns to that page. A
  link to anything else opens it: another wordplay on the site in place of
  this one, with the way back over its title; anything off the site in a
  new tab. A wordplay is linked by its name, `<a href="sodium-2">`, which
  resolves beside this one.
- Chats in the page (`data-chat` on a `<div>`), so a question is asked
  where it arises. A chat in the page is one conversation with the tutor's,
  shown in it as a thread: the tutor reads every message in order, knowing
  where it was said, and answers there. A chat holding an output the tutor
  writes is drawn by the teacher: Wordplay draws no box in it.
- Mathematics: `$…$` in text is a formula and `$$…$$` one on its own line,
  drawn with KaTeX, except inside `code`, `pre`, `script`, `style` and
  fields. The tutor reads the source as written.
- A calculator for number answers, checked by the component's script.
- A step or reveal given carries `data-met`, its id, and fires a
  `wordplay:met` event as it appears and again as the page is drawn, the
  id in `event.detail.id`, for the teacher's CSS
  (`body:has(#solved) .hint { display: none }`) or scripts ("Scripts").
- A wall every reader of a wordplay shares, written by the page's scripts
  ("Sharing", below).
- Limits on size, and checks before a wordplay is published.

Its attributes:

| Attribute | On any element, means |
|---|---|
| `data-chat` | On the `<body>`: the tutor's chat beside the page. On a `<div>`: a chat with the tutor drawn in it, after what it holds, named by its accessible name; context inside it is given with it. The tutor answers in each chat and writes where the teacher marked; what has no chat to be said in is said nowhere. |
| `data-progress` | On the `<body>`: progress drawn over the page. |
| `data-pages` | On the `<body>`: the sections directly in it are pages, one shown at a time. |
| `data-skip` | On the `<body>`: Skip at each waiting step. On a step, `data-skip="false"`: none for it, and the student must earn it. |
| `data-calculator` | On an `<input>`: the box has the calculator, which takes its place, the box's value what it works out (`86.4 s`). Its accessible name is its label. On a chat: one under it, `=` adding to the message. Its keys are the `data-key` elements in the chat, or in the input's `<label>`. |
| `data-key="…"` | On a `<data>`: a key on that calculator, never shown in the page. `number` (its text: `<data data-key="number">24 h</data>`), `constant` (its text the name, its `value` the value: `<data data-key="constant" value="9.81 m/s^2">g</data>`), `unit`, or `function` (a key from the calculator's library, by name). |
| `data-person="…"` | Filled with the profile of the reader of an entry on the wall: `mine`, or an entry's place in `entries`, from 0 ("Sharing"). |
| `data-opening="…"` | On the `<body>`, the tutor's first line in its chat beside the page; on a chat, its first line in that chat. Shown before anything is said, Markdown and math as in any reply, and the tutor is told it. Without it, the host's own line. |

**The name of a step**, in the chat, on the page and in the creator, is its
accessible name: its `aria-label`, else its first heading, else its id.

**A key with a condition** is put on the calculator once it holds, named
`k1`, `k2`, … when it has no `id`. It counts for nothing, and has no Skip.

**The keys** a calculator may hold from its library, by name:

{{keys}}

**Sharing.** Each reader has one entry on the wordplay's wall, which the
page's script writes, and every reader reads the whole wall:

| Member | Does | At most |
|---|---|---|
| `share(value)` | This reader's entry, any JSON, replacing their last; `null` takes it away. A value the block below refuses is not sent, with a warning in the console. | {{maxShared}} bytes, {{maxShares}} a minute |
| `shared` | The wall, `{ mine, entries, count, tally }`: this reader's entry, every reader's newest first ({{sharedEntries}} unless the block says), how many in all, and `tally`, for each field the block names: `{ [value]: n }` for a choice or boolean field, `{ n, sum }` for a number field, over every entry and not only those sent. An entry is `{ value, at, guest, by?, who?, facts? }`. There before the first script runs. | |
| `person(who)` | The site's chip for an entry's `who`, or the anonymous one for `null`. | |

**`wordplay:shared`** fires on the document when the wall changes. An
element marked `data-person` is filled with its entry's chip.

What the page shares is the reader's claim, since the page runs in their
browser. `by: "tutor"` (the tutor's judgement) and `facts` (the server's
count of the attempt) are not: rank by those where fairness matters.

One optional `<script type="application/wordplay-shared">` bounds an entry,
as JSON, every key optional:

| Key | Means |
|---|---|
| `fields` | The entry is an object of these fields only: `{ "type": "number", "min", "max" }`, `{ "type": "choice", "of": [...] }`, `{ "type": "boolean" }` or `{ "type": "text", "max" }` (at most {{maxSharedText}}) |
| `names` | `true`: each entry carries `who`, its reader's name, handle and picture, `null` for a guest or a reader anonymous by choice |
| `guests` | `false`: a guest of a device may not share |
| `entries` | How many are sent, at most {{maxSharedEntries}} |
| `facts` | Any of `"seconds"` (start to done), `"steps"` (the tutor gave), `"messages"`, `"skips"` |
| `tutor` | `true`: the tutor may write a reader's entry as it judges it |
| `tally` | The names of fields to total over every entry: for a `choice` or `boolean` field, how many entries hold each value; for a `number` field, how many and their sum. A `text` field cannot be tallied |

The class's guesses, shown once the student has guessed:

```html
<body data-wordplay="1" data-chat>
  <script type="application/wordplay-shared">
    { "fields": { "guess": { "type": "number", "min": 0, "max": 100 } } }
  </script>
  <label>Your guess <input id="guess" type="number"></label>
  <button id="send">Send</button>
  <p id="guessed" data-expect="The student has guessed">The class: <output id="class"></output></p>
  <script>
    const draw = () => {
      const all = (document.wordplay?.shared?.entries ?? []).map((one) => one.value.guess);
      const out = document.querySelector("#class");
      if (out) out.textContent = all.join(", ");
    };
    document.querySelector("#send").onclick = () => {
      document.wordplay?.share?.({ guess: Number(document.querySelector("#guess").value) });
      document.wordplay?.met("guessed");
    };
    document.addEventListener("wordplay:shared", draw);
    document.addEventListener("wordplay:met", draw);
  </script>
</body>
```

**Standing.** A wordplay reads where its reader stands on the wordplays it
links to, and on itself:

| Member | Does |
|---|---|
| `standing(name?)` | For a wordplay this file links to, by the name its link uses (`<a href="sodium-2">`), `{ title, completed, minutes, mine }`: its title; how many readers finished it, each once; its length in minutes as the site says it, `null` until measured; and `mine`, this reader's own, `null` until they start it, else `{ done, given, total }`. With no name, this wordplay's own. A name the file does not link to answers `null`. There before the first script runs, and not changed while the page is open. |

A link to a wordplay is marked by the host as its reader stands on it:
`data-started` while a conversation of theirs is under way, `data-done` once
any of theirs finished it, so a page colors its links in CSS alone. A guest
of a device has `mine` for this device's conversations. Another host answers
zero counts and `null` for every name.

**Checks.** Wordplay says what looks like a mistake: a step with no
accessible name, an id used twice, a chat unnamed, named as another or for
its box ("Chat about …"), a key outside a calculator or of an unknown kind,
an output the tutor writes unnamed, named as another, named with a colon or
inside context, `data-tutor` on anything but an `<output>`, an attribute
nearly one of these or no longer read, `document.tutor`, the object's old
name, in a script, `data-opening` on anything but the body or a chat or
over {{maxOpening}} characters, a declaration on the body with a value or
on anything but the body, `data-skip` on anything but the body or a step or
with any value but "false" on a step, `data-skip="false"` in a wordplay
that offers no Skip, an opening line on the body of a wordplay with no chat
beside the page, a condition in a wordplay with no chat anywhere, and a
shared block it cannot read. It refuses to publish an unknown
`data-wordplay` version, a file over the limit, an id used twice, a shared
block it cannot read, and a control the tutor cannot see: one with no
accessible name, on a page where nothing is named or live.

## Limits

| What | At most |
|---|---|
| The file | {{maxFile}} characters |
| A condition | {{maxCondition}} characters |
| One live announcement, or one `tell` | {{maxAnnouncement}} characters |
| One line the tutor writes | {{maxWritten}} characters |
| One `say` | {{maxSaid}} characters |
| An opening line (`data-opening`) | {{maxOpening}} characters |
| What a page keeps (`keep`) | {{maxKept}} bytes |
| One entry shared (`share`) | {{maxShared}} bytes, {{maxShares}} a minute |
| A shared text field | {{maxSharedText}} characters |
| Entries sent | {{maxSharedEntries}} |
| Keys in one group of a calculator | {{maxKeys}} |

## Versions

`data-wordplay="1"` is redefined by this document. The format is a
prototype: a form it drops is no longer read, and published wordplays are
written again. There is no custom element. The extension stays `.wplay`:
HTML to run with a tutor; editors highlight it as HTML.

## Words

One word per thing, in the file, the site, the prompts and the docs.

| Word | Means |
|---|---|
| wordplay | One `.wplay` file, what a reader works through |
| student | The person working through it |
| teacher | Who wrote it |
| tutor | What the student talks to |
| host | What runs the file: Wordplay's site, or another |
| step | A `<section>` the student sees with a condition; it counts |
| reveal | Anything else the student sees with a condition; it does not |
| context | An element only the tutor reads |
| chat | The tutor's chat beside the page, or one in it (`data-chat`) |
| expect | The condition, the moment as the teacher wrote it |
| given | Shown to the student, or, for context, read by the tutor |
| waiting | Not given, its parent given: its condition is being judged |
| progress | How many steps are given, of the steps |
| done | Every step given |
| skip | Giving one waiting step without its condition |
