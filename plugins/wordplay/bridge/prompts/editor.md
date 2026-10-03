{{!-- The ground truth is the editor itself: `WritingHelp`, lib/markdown.ts
      and the component kit (`@aeneas/format/components`). Asked for after
      the assistant wrote a diagram that would not draw: "It should have that
      context on how to properly format things for it to accept it."
      Figures are SVG since 1 October (Javi: "do svg figures. I'll see if
      it's worth adding tikz later on"); the TikZ notes were in git before. --}}
How to write for this editor; anything else shows in red or as plain
source:

- **Text** is Markdown: headings, lists (bulleted or numbered), tables,
  quotes, emphasis, inline code and links. No HTML.
- **Maths** is KaTeX: `$…$` inline, `$$…$$` displayed. `aligned`,
  `pmatrix` and `cases` work inside the maths; `\R \N \Z \Q \C \eps` are
  defined; a `\newcommand` anywhere applies to the whole text. No theorem
  environments, `\label`, `\ref` or `\cref`; `\tag{3.1}` works.
- **Figures** are SVG in a custom interaction that asks nothing (A
  figure, above): one `<svg>` with a `viewBox` and no width or height, so
  it fits any page, `role="img"` and an `aria-label` saying what it shows,
  labels as `<text>`. No TikZ: Wordplay does not draw it. **Its look**: lines
  and labels in `currentColor`, the page's ink, light or dark, and
  `var(--wp-accent)` for the one thing the figure is about. A colour is for
  what it means in the subject, from the page's own: `var(--wp-success)`,
  `var(--wp-danger)`, `var(--wp-ink-muted)`. Leave shapes unfilled, or
  tint them with `fill-opacity="0.15"`; never a white or pale fill, which
  hides a label in the dark.
- **Pictures** cannot be drawn: ask the author for one and place it by its
  address, `![A caption](address)`. Where the editor has an upload, they
  can add a photograph or scan there.
