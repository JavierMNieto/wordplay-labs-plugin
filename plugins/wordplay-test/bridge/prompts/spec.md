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
https://forum.wordplaylabs.com/p/cmuun53g200122ppbn1eb314m, simplified on
5 October, and on 8 October, after the company meeting of 7 October, to
data-expect and data-context, with document.tutor for scripts and sections
as the steps that count. The decisions behind it, and its history, are in
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

A `.wplay` file is an HTML document with three attributes on it. ARIA added
attributes to HTML so a screen reader could read a page; these three add
what a tutor needs on top of that.

| Attribute | On any element, means |
|---|---|
| `data-wordplay="1"` | On `<body>`: this document is a wordplay. |
| `data-context` | The student never sees this element. The tutor reads it, and operates it if it is a control. |
| `data-expect="…"` | This element is given once a condition holds, judged by the tutor from the conversation, or said by the page's script. |

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
recognize it, judged by the tutor or said by a script (`document.tutor.met`).

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

**Privacy.** Every element marked `data-context` is stripped before anything
reaches the student's browser. This is the one rule a host must never break.
A step or reveal reaches the browser only once given, without its
`data-expect`, which is often the answer.

**Unknowns.** A host ignores an attribute it does not know and shows the
element as HTML.

**What the tutor is given at any moment.** The tree of what is shown, its
announcements, and what the page told it since its last turn. The context
given so far. Each part not yet given whose parent has been, by id and
condition, so it knows what to watch for. The steps the student skipped.
Where a student is is never in the file; it is worked out from their
conversation. With no conversation, a host shows the whole wordplay.

## Scripts

The teacher's scripts run as on any page. Before any runs, the host puts
one object on the document, `document.tutor`, with two members:

| Member | Does |
|---|---|
| `met(id)` | That element's condition holds: the host gives it, as if the tutor had said so. An id not waiting is ignored. |
| `tell(text)` | The text reaches the tutor at its next turn, cut to one announcement. A live region is read out to the student too; this is not. |

Opened as plain HTML a page has no `document.tutor`, so a script calls
`document.tutor?.met("solved")`. A script runs in the student's browser,
which can read it: a secret goes in `data-context` or behind a
`data-expect`, never in a script.

## Progress

**Progress** is the steps given over the steps in the file, nested ones
counted flat; it says how many, never in what order, since steps do not
wait for one another. **Done** is every step given; a wordplay with no step
is never done. **Skip**, where a host offers it, gives one waiting step as
if its condition held, and nothing else; what is inside it then waits as
usual. A step with `data-skip="false"` has none, and the student must earn
it. A step skipped counts, and the tutor is told of it.

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

  <aside id="stuck" data-expect="they have lost three games in a row">
    <p>Try a smaller game first: who wins with 4 matches on your turn?</p>
  </aside>
  <script>
    // The game calls gameOver at the end of each game.
    let lost = 0;
    function gameOver(won) {
      lost = won ? 0 : lost + 1;
      if (lost === 3) document.tutor?.met("stuck");
      if (won) document.tutor?.tell("They beat the computer from 21 matches.");
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

Opened as HTML, it is the lesson without the tutor. As a tree, conditions
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
ignore, and the file loses nothing:

- Each `<section>` directly in the body without a condition is drawn as a
  page, with the pages along the foot; one with a condition is a step,
  drawn where it is.
- Progress as a thin bar over the page that only shows it, and on the card
  of a lesson started; Skip at each waiting step, where it will appear.
- The tutor's chat beside the page, and chats in the page (`data-chat`),
  so a question is asked where it arises. A chat in the page is one
  conversation with the tutor's, shown in it as a thread: the tutor reads
  every message in order, knowing where it was said, and answers there.
- Mathematics: `$…$` in text is a formula and `$$…$$` one on its own line,
  drawn with KaTeX, except inside `code`, `pre`, `script`, `style` and
  fields. The tutor reads the source as written.
- A calculator for number answers, checked by the component's script.
- A step or reveal given carries `data-met`, its id, and fires a
  `wordplay:met` event as it appears, the id in `event.detail.id`, for the
  teacher's CSS (`body:has(#solved) .hint { display: none }`) or scripts.
- Limits on size, and checks before a wordplay is published.

Its attributes:

| Attribute | On any element, means |
|---|---|
| `data-chat` | On a `<div>`: a chat with the tutor drawn in it, after what it holds, named by its accessible name. Context inside it is given with it, as any context is with its parent. |
| `data-calculator` | On an `<input>`: the box has the calculator. Its accessible name is its label. On a chat: the calculator beside it. Its keys are the `data-key` elements in the chat, or in the input's `<label>`. |
| `data-key="…"` | On a `<data>`: a key on that calculator, never shown in the page. `number` (its text: `<data data-key="number">24 h</data>`), `constant` (its text the name, its `value` the value: `<data data-key="constant" value="9.81 m/s^2">g</data>`), `unit`, or `function` (a key from the calculator's library, by name). |
| `data-skip="false"` | On a step: no Skip for it. |

**The name of a step**, in the chat, on the page and in the creator, is its
accessible name: its `aria-label`, else its first heading, else its id.

**A key with a condition** is put on the calculator once it holds, named
`k1`, `k2`, … when it has no `id`. It counts for nothing, and has no Skip.

**The keys** a calculator may hold from its library, by name:

{{keys}}

**Checks.** Wordplay says what looks like a mistake: a step with no
accessible name, `data-skip` off a step, an id used twice, a chat unnamed
or named as another, a key outside a calculator or of an unknown kind, and
an attribute nearly one of these or no longer read. It refuses to publish
an unknown `data-wordplay` version, a file over the limit, an id used
twice, and a control the tutor cannot see: one with no accessible name, on
a page where nothing is named or live.

## Limits

| What | At most |
|---|---|
| The file | {{maxFile}} characters |
| A condition | {{maxCondition}} characters |
| One live announcement, or one `tell` | {{maxAnnouncement}} characters |
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
| wordplay | One lesson, one `.wplay` file |
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
