# Wordplay for Claude

[Wordplay](https://wordplaylabs.com) is where people publish lessons that a
learner works through in conversation with a tutor, which has the answer and
will not give it away. A lesson is one text file, a `.wplay` file.

This plugin teaches Claude to write one with you from your own material (a
book, your notes, a conversation in which you worked something out), lets you
try it with the tutor before anyone else sees it, and publishes it on Wordplay
when you ask.

## Install

**In Claude Code**, in your terminal:

```
claude plugin marketplace add JavierMNieto/wordplay-labs-plugin
claude plugin install wordplay@wordplay
```

Then run `/reload-plugins` and ask: "Help me write a Wordplay lesson from my
notes." For a whole session spent building, start Claude Code with
`claude --agent wordplay:builder`.

**In the Claude app** (chat or Cowork): open
[Customize, then Plugins](https://claude.ai/customize/plugins), choose Add,
then Add marketplace, and paste `JavierMNieto/wordplay-labs-plugin`. Add
Wordplay, then connect it on its Connectors tab, which signs you in to
Wordplay.

A plugin added in the app is saved to your account, so it is in Claude Code
and the desktop app too.

## What it holds

- **The `write-a-lesson` skill**: how to write a lesson, step by step, as one
  `.wplay` file kept in your folder.
- **The builder agent**, for a session spent writing one with you.
- **`/wordplay:trial`**, which sends a simulated learner through your lesson
  and reports where they got stuck.
- **Wordplay's creator on your computer**, to see the lesson as a learner does
  and try it with the tutor.
- **Wordplay's MCP server** (`https://wordplaylabs.com/api/mcp`), which reads
  lessons and publishes yours. Nothing is published unless you ask.

What a `.wplay` file can hold, every block and field:
[wordplaylabs.com/format](https://wordplaylabs.com/format).

## Where it comes from

This repository holds the built plugin, published from Wordplay's own
repository each time the site is released; changes made here are replaced.
For a problem or a question, open an issue.

MIT licensed: see [LICENSE](LICENSE).
