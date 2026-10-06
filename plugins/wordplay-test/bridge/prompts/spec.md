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
https://forum.wordplaylabs.com/p/cmuun53g200122ppbn1eb314m, and simplified
on 5 October (Javi: "we only need data verify, data expect, and data
context"). The decisions behind it, and its history, are in
docs/wordplay-format.md.
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
attributes to HTML so a screen reader could read a page; these four add what
a tutor needs on top of that.

| Attribute | On any element, means |
|---|---|
| `data-wordplay="1"` | On `<body>`: this document is a wordplay. |
| `data-context` | The student never sees this element. The tutor reads it, and operates it if it is a control. |
| `data-verify="…"` | This element is given once a condition holds, checked by the host against the page. |
| `data-expect="…"` | This element is given once a condition holds, judged by the tutor from the conversation. |

Everything else is HTML, written to ARIA, and is shown as HTML.

## Rules

**One rule.** An element with a condition is given once its condition
holds: to the student, or, if it is marked `data-context`, to the tutor.
Until then it is not given at all, with everything it holds. An element
without a condition is given with its parent. The body is always shown.

**Steps.** An element the student sees that carries a condition is a
step. Nothing marks a step done: what comes before a step is done when the
step appears, and the wordplay is done when its last step does. So a
condition goes on what the student should see next, written as the moment
that earns it.

**Order.** A condition is judged once its element's parent has been given.
Conditions do not wait for one another otherwise, so a hint the student
never needs blocks nothing after it. Two steps that must come one after the
other nest, or say in their conditions what they follow.

**Context.** A context element holds whatever the teacher wants the tutor
to read; the tutor is given its text, with inline markup kept as written,
so math and code survive. Context with no condition is given as soon as its
parent is; with one, once the condition holds. Context nests by nesting.

**The two conditions.** `data-verify` is `name = value`, where `name` is
the accessible name of an element on the page and `value` is its text or
state; `and` and `or` combine them. `data-expect` is prose, written the way
the teacher would recognize the moment. An element carries at most one of
the two.

**Ids.** The host names each step and each context element by its own `id`
when it has one, and otherwise numbers them in document order: steps `s1`,
`s2`, … and context `c1`, `c2`, …. The tutor says which condition holds by
that id.

**Reading the page.** The tutor is given the page as a screen reader would
read it: the accessibility tree (role, name, state) of what is shown, and
what its live regions have announced. Nothing about how the page is drawn
reaches it. A page that works for a blind student works for the tutor.

**Acting on the page.** Anything a student could do with a keyboard, the
tutor can do by accessible name: press a button, set a field, choose an
option. An element marked `data-context` is removed from the student's page
and their screen reader, and placed in the tutor's tree alone.

**Privacy.** Every element marked `data-context` is stripped before anything
reaches the student's browser. This is the one rule a host must never break.

**Unknowns.** A host ignores an attribute it does not know and shows the
element as HTML.

**What the tutor is given at any moment.** The tree of what is shown, and
its announcements. The context given so far. And for each step and context
element not yet given whose parent has been, its id and its condition, so it
knows what the teacher anticipated and what to watch for. Where a student is
is never in the file; it is worked out from their conversation. With no
conversation, a host shows the whole wordplay.

## Example

Page 2 of *The game from Marienbad*:

```html
<!doctype html>
<html lang="en">
<body data-wordplay="1">

<div data-context>The game is Nim with one row. The computer plays perfectly.</div>

<section aria-label="One row, small steps">
  <p>There is only one row of 21 matches. On your turn you take 1, 2 or 3. The
  player who takes the last match wins. You go first. Play a few games, look
  for a pattern, and tell your tutor your idea.</p>

  <div class="matches" role="group" aria-label="21 matches. Take 1, 2 or 3.">…</div>
  <p aria-live="polite">Your move.</p>
  <button>Start again</button>
  <form aria-label="Set up a position" data-context>…</form>

  <div data-context>A student stuck here: who wins with 4 matches on your turn?</div>
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

    <section aria-label="Fort Boyard solved"
      data-expect="they say the candidate cannot win from 21">
      <p>Right: 21 is one more than a multiple of 4, so the Master of Time
      always wins.</p>
    </section>
  </section>
</section>

</body>
</html>
```

Opened as HTML, it is the lesson without the tutor. As a tree, conditions
on the edges:

```
context c1  (always)
context c2  (always: the set-up form)
context c3  (always: a student stuck here)
context c4  ── expect: they say the trick is to always take 1
context c5  ── expect: they ask why 4 is the bad number
└── context c6  ── expect: they ask whether it works when the last stick loses
step "One row solved"  ── expect: they say, in their own words, always leave a multiple of 4
└── step "Fort Boyard solved"  ── expect: they say the candidate cannot win from 21
```

## What Wordplay adds

The above is the standard. Wordplay's site adds tooling that another host may
ignore, and the file loses nothing:

- Each `<section>` directly in the body is drawn as a page, with the pages
  along the foot. One with a condition is a page that appears when it holds.
- The tutor's chat beside the page, and chats in the page (`data-chat`),
  so a question is asked where it arises. A chat in the page is one
  conversation with the tutor's: the tutor reads every message in order,
  knowing which chat it was said in, and answers there. The tutor's chat
  shows each chat in the page as a thread where it began.
- Mathematics: `$…$` in text is a formula and `$$…$$` one on its own line,
  drawn with KaTeX, except inside `code`, `pre`, `script`, `style` and
  fields. The tutor reads the source as written.
- A calculator for number answers, and `≈` with a tolerance in `data-verify`.
- A student may skip waiting for a step, unless the step says not.
- The student's page never carries a condition: a step given goes without
  its `data-expect` or `data-verify`, which is often the answer.
- A step whose condition has held carries `data-met`, its id, in the
  student's page, and the page fires a `wordplay:met` event on it as it
  appears, the id in `event.detail.id`. What follows a step listens for it there: the
  teacher's CSS (`body:has(#solved) .hint { display: none }`, since a step
  is not in the page at all before it is met), their scripts, or another
  condition (`shown`, below). A condition is written once.
- Limits on size, and checks before a wordplay is published.

Its attributes:

| Attribute | On any element, means |
|---|---|
| `data-chat` | On a `<div>`: a chat with the tutor drawn in it, after what it holds, named by its accessible name. Context inside it is given with it, as any context is with its parent. |
| `data-calculator` | On an `<input>`: the box has the calculator. Its accessible name is its label. On a chat: the calculator beside it. Its keys are the `data-key` elements in the chat, or in the input's `<label>`. |
| `data-key="…"` | On a `<data>`: a key on that calculator, never shown in the page. `number` (its text: `<data data-key="number">24 h</data>`), `constant` (its text the name, its `value` the value: `<data data-key="constant" value="9.81 m/s^2">g</data>`), `unit`, or `function` (a key from the calculator's library, by name). |
| `data-skip="false"` | On a step: the student may not skip waiting for it. Without it, Skip shows the first step waiting on the page. |

**`≈` in `data-verify`.** `name ≈ value ± n%` holds when the element's value,
read as a number with its unit, is within `n` percent of `value`:
`answer ≈ 86.4 s ± 1%`. Without `± n%` the tolerance is 0.1%.

**Writing a condition.** `data-verify` is one or more `name = value`
comparisons joined by `and` and `or` (`and` binds tighter), with parentheses
to group. A name matches ignoring case and spacing. A value that holds `and`,
`or`, `=` or a parenthesis is written in double quotes. `checked = The slower
clock` holds when the element named *The slower clock* is checked, and
`pressed`, `selected` and `expanded` work the same way. `shown = One row
solved` holds once an element of that name is on the page, so a condition
can follow another without restating it.

**The name of a step**, on the page and in the creator, is its accessible
name: its `aria-label`, else its first heading, else its id.

**A key with a condition** is put on the calculator once the condition
holds, as a step is given, and is named `k1`, `k2`, … in document order when
it has no `id`, for the tutor's `[[SHOW]]`. It is not a step: nothing waits
for it, it is never skipped to, and the wordplay is done without it.

**The keys** a calculator may hold from its library, by name:

{{keys}}

**Checks.** Wordplay says what looks like a mistake as it reads a wordplay:
a `data-verify` that names nothing on the page, an element with both
conditions, an id used twice, a chat with no name or the name of another,
a key outside a calculator or of a kind it does not know, and an attribute
that is nearly one of these.
It refuses to publish a wordplay whose `data-wordplay` version it does not
know, one over the limit, one with a `data-verify` that does not read or an
id used twice (either leaves a step that never appears), and one with a
control the tutor cannot see: a button, field or canvas with no accessible
name, on a page where nothing is named and nothing is live.

## Limits

| What | At most |
|---|---|
| The file | {{maxFile}} characters |
| A condition | {{maxCondition}} characters |
| One live announcement | {{maxAnnouncement}} characters |
| Keys in one group of a calculator | {{maxKeys}} |

## Versions

`data-wordplay="1"` is redefined by this document. The format is a
prototype: a form it drops is no longer read, and every published wordplay is
written again in the current form. Version 1 as first written, Markdown with
colon blocks, a line dividing the tutor's half, an answers list and a
scripting interface for components, is gone, and so is the step attribute of
4 October: a condition is all a step needs. There is no custom element. The
extension stays `.wplay`: the content is HTML, the extension says to run it
with a tutor. Editors should highlight it as HTML.

## Words

One word per thing, in the file, the site, the prompts and the docs.

| Word | Means |
|---|---|
| wordplay | One lesson, one `.wplay` file |
| student | The person working through it |
| teacher | Who wrote it |
| tutor | What the student talks to |
| host | What runs the file: Wordplay's site, or another |
| step | An element the student sees that carries a condition; it appears when the condition holds |
| context | An element only the tutor reads |
| chat | Where the student talks to the tutor: the tutor's chat beside the page, or one in the page (`data-chat`) |
| verify | A condition the host checks against the page |
| expect | A condition the tutor judges from the conversation |
| given | Shown to the student, or, for context, read by the tutor |
| waiting | Not yet given, with its parent given: its condition is being judged |
