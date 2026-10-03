# Wordplay, the plugin

Writing lessons from Claude Code, each kept as one `.wplay` file in your
own folder, tried there with the tutor, and published on Wordplay
when you say so. The format is `docs/wordplay-format.md` in the site's
repository. The plugin is Wordplay's (Javi, 30 September); the Wordplay Forum
has an MCP server of its own, for reading threads and writing drafts.

What it holds:

- **Wordplay's MCP server** (`.mcp.json`), which Claude Code connects to
  and signs you in to the first time, in your browser: `search` and
  `get_lesson` for reading lessons, `list_lessons` for your own, and
  `save_lesson`, which publishes a lesson on Wordplay under your name and
  changes one you published before. Ask for it in words; it publishes.
- **The preview** (`bridge/wordplay.mjs`, a second MCP server Claude Code
  starts with Node): Wordplay's creator on a lesson in your folder, on
  `http://127.0.0.1:4747/preview/local`, with its assistant, the tutor and
  Simulate running on your own Claude Code or key. See below.
- **The `write-a-lesson` skill**, generated from the prompts in the site's
  repository (`scripts/skills.ts`; edit the prompts, never this file).
- **`/wordplay:trial` and the `learner` agent**: simulated readers playing a
  lesson through against its tutor, and a report of what a machine can
  check. See below.
- **The `builder` agent**: a lesson written with you, in your own Claude
  Code. See below.
- **The player** (`player/`, bundled from the site's own components): the
  same lesson page, tutor and editor as a page of its own, which the
  builder publishes as a claude.ai artifact with the lesson beside it, so
  an author in claude.ai tries it with the tutor on their own plan.

## Two plugins: the site's and the test site's

Which site you write to is which plugin you run, never a setting. Both come
from [wordplay-labs-plugin](https://github.com/JavierMNieto/wordplay-labs-plugin),
each in a marketplace of its own:

| Plugin | Site | Marketplace | Its commands |
|---|---|---|---|
| `wordplay` | https://wordplaylabs.com | `wordplay`, the repository's `main` | `/wordplay:…` |
| `wordplay-test` | https://test.wordplaylabs.com | `wordplay-test`, its `test` branch | `/wordplay-test:…` |

```
claude plugin marketplace add JavierMNieto/wordplay-labs-plugin
claude plugin install wordplay@wordplay

claude plugin marketplace add JavierMNieto/wordplay-labs-plugin#test
claude plugin install wordplay-test@wordplay-test
```

A change reaches `wordplay-test` with the test site, and `wordplay` with
the site, so a plugin and its site are the same build. Try a change on the
test site's plugin first; both can be installed at once.
`/plugin marketplace update wordplay` (or `wordplay-test`) brings in what
was published since: the plugins have no `version` on purpose, so an
update follows the latest build rather than waiting for a number to
change. Each plugin signs you in to its own site the first time its tools
are used.

`WORDPLAY_URL` still points either plugin's preview at a site of your own.
The site's connector is always the plugin's own site: claude.ai and Cowork
take a connector's address as it is written, with no variables in it.

**If you already connected the site as a connector in claude.ai**, its tools
are there twice, once under each name. Keep one: disconnect the connector,
or leave the plugin's server off in `/mcp`.

## The builder

The builder writes a lesson with you the way the site's lesson assistant
used to: one decision at a time, a form for each choice, the lesson as a
`.wplay` file in your folder with the preview beside it, and an offer,
now and then, of something the lesson could use (a step, a reveal, a
number the calculator checks, a diagram). Its rules are one text,
`src/prompts/lesson-guide.md` in the site's repository, rendered three
ways by `npm run skills`:

- **A whole session as the builder**: `claude --agent wordplay:builder`
  in the folder the lesson lives in. Every turn follows its rules, and it
  asks with Claude Code's question form.
- **In a chat you already have**: `/wordplay:write-a-lesson`, or just ask
  for a lesson and the skill is picked up. The same rules, in a session
  that also does other things.
- **In claude.ai**: upload `claude-ai/write-a-lesson` (zipped) as a skill
  in claude.ai's settings, or paste its `SKILL.md` into a Project's
  instructions. It asks with claude.ai's own choices and keeps the lesson
  in the player page it publishes in the chat, which offers "Download the
  file".

**A rehearsal, to check it still behaves** after a change to the rules:
start `claude --agent wordplay-test:builder` in an empty folder with a
subject you know, and look for these.

1. The first turn writes the title and the opening only, and asks one
   form.
2. No turn writes two steps.
3. No answer is set that neither your material nor you gave it.
4. Nothing of the tutor's half (answers, notes) is above the `:::tutor`
   line.
5. It offered something from the catalogue at least once, and never the
   same thing twice unasked.

## The preview

Ask Claude Code to preview a lesson ("preview light-second"), or it offers
to after writing one. Its `preview_lesson` tool answers with an address;
open it in a browser, or in VS Code's Simple Browser beside the file. You
try the lesson as a learner:

- **The page is Wordplay's creator**, the same one as on the site, bundled in
  the plugin's `player/` folder (`creator.js`): the lesson as a document,
  the side panel's Tutor, Map, Calculator and Simulate, the assistant's
  chat, and the student's view with the tutor. The lesson is your file:
  what you type is saved to it as you type, and what Claude Code writes to
  it arrives in the page tinted, as the assistant's changes do. If you were
  both changing the same lines, the file's version is kept and undo brings
  yours back.
- **The assistant, the tutor and Simulate run on your Claude Code**
  (`claude -p`), on your own plan, with the prompts Wordplay sends and nothing
  else: no tools, and none of your settings, memory, skills or MCP
  servers. **Each chat has its model by Send**: the assistant's and the
  tutor's start on the ones Wordplay runs (`/api/wordplay` says which), and
  the one you pick is kept in your browser; Simulate runs on the tutor you
  picked. `WORDPLAY_BUILDER_MODEL` and `WORDPLAY_BUILDER_EFFORT`, and
  `WORDPLAY_TUTOR_MODEL` and `WORDPLAY_TUTOR_EFFORT`, set where each starts
  instead.
- **What only Wordplay keeps is not here**: Your lessons, version history, the
  bin, attached files and pictures. **Publish is Claude Code's**: the
  Publish tab says what to ask it, and it publishes through Wordplay's MCP
  server above, under your account.
- **In the Claude desktop app's Code tab**, Claude can show the preview in
  the Browser pane beside the chat, and look at the page and use it as a
  reader would. `preview_lesson` says how: the pane attaches to a running
  server by its bare address, `http://127.0.0.1:4747`, which leads to the
  preview.
- **No API key, ever**: the tutor runs on your own Claude Code and your
  plan, and publishing signs you in through your browser. The plugin passes
  no key or token to anything (2 October, for Anthropic's directory).
- **Numbers are checked on your machine** against the answers in the file, and the
  answers never leave it except inside the tutor's prompt.
- **Each conversation is a file**, `.trials/<name>/<id>.json` beside the
  lesson, which Claude reads with `read_trials`. Add `.trials/` to your
  `.gitignore` if you would rather not commit them.
- **Diagrams are drawn on your machine**, with the site's own TeX engine
  and code, so each one looks as readers will see it. The engine is about
  90 MB, so it is installed once, the first time a lesson has a diagram,
  into `~/.cache/wordplay/engine` with npm (about ten seconds), and drawn
  diagrams are kept in `~/.cache/wordplay/tikz`. `WORDPLAY_CACHE_DIR`
  moves both. Only if that install cannot happen (no npm, or offline the
  first time) are they left undrawn in the preview, and drawn on the site
  when the lesson is published.

## An automated trial

`/wordplay:trial path/to/lesson.wplay` plays the lesson through with
simulated readers (add "with Opus", say, to run them on another tutor): `learner` subagents, one per reader, each shown only the
page (never the tutor's half of the file) and talking to the tutor through the preview's
`start_trial`, `say_to_tutor`, `answer_step` (a step's own answer box) and `enter_number`. By default there are two:
one who works straight through, and one who gets stuck, gets a unit wrong
and asks for the answer outright. Describe your own after the path.

Then `trial_report` reads each trial mechanically: what appeared on the page
and when, whether and when the lesson completed, and flags for what a
machine sees better than you do. Did the tutor say an answer, or what a
hidden part holds, before the reader got there? Did it write a marker the
page could not act on, or ask the calculator for a value the lesson never
declared? It never judges the teaching; whether a reader earned it is
yours to feel, in a trial of your own. It also says what the trial's
replies used in tokens, from the tutor's own count, which is what your plan
or key paid for it. The trials are files in `.trials`
like yours, so `read_trials` reads them and the preview's history lists
them. They run on your plan, the readers and the tutor alike.

The preview needs Node 20 or later, and npm, on your path.
