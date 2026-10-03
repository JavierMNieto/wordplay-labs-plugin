{{!-- The creator's builder is sent the builder's guide (lesson-guide.md,
      for the `creator` host), what the author attached, their other lessons
      and their latest conversation with the tutor (trial.md), and then
      this: the lesson as the author's page holds it now, sent with every
      message. It is the last of the cacheable pieces, so everything above it
      is read from cache while this changes (`builderPrompt` in
      lib/builder-turn.ts, docs/creator.md). --}}
# The lesson as it stands

{{#if file}}
`````wordplay
{{file}}
`````
{{else}}
Nothing is written yet. Your first reply writes the whole file in a
`<file>` block: the front matter, the title and the opening, and the
`:::tutor` line with no answers under it yet.
{{/if}}

{{#if page}}
# Where the author is

The lesson is in pages, and the author has {{page}} open in front of them.
"Here" and "this page" mean that page, and a change they ask for without
saying where belongs on it.

{{/if}}
# What the page says is wrong with it

{{#if problems}}
{{#each problems}}
- {{.}}
{{/each}}
{{else}}
Nothing.
{{/if}}
{{#if changes}}

# What became of your last changes

{{#each changes}}
- {{.}}
{{/each}}
{{/if}}
{{#if fresh}}

# Since your last reply

The author has tried the lesson with the tutor: their conversation is
above the lesson, and is new to you.
{{/if}}
