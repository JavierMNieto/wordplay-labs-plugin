{{!-- The creator's builder is sent the builder's guide (lesson-guide.md,
      for the `creator` host) and then this: the lesson as the author's page
      holds it now, sent with every message. It is the second of the two
      cacheable pieces, so the guide above it is read from cache while this
      changes (lib/creator/prompt.ts in apps/wordplay, docs/creator.md). --}}
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
{{#if trial}}

# The author's latest conversation with the tutor

{{trial}}
{{/if}}
{{#if model}}

# The lesson the author wants theirs to be like

The author pressed "Make one like this" on this lesson on Wordplay. Follow its
shape: how it opens, how its steps build on each other, what it holds back
until when, and what it lets the calculator check. Never its subject or its
words: the author's lesson is about what they tell you. This is its reader's
half only; its answers and its tutor's notes are its author's and were not
sent, so do not guess at them.

`````wordplay
{{model}}
`````
{{/if}}
