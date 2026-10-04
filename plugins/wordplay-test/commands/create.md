---
description: Open Wordplay's creator in the browser, on a lesson in this folder or a new one
argument-hint: [a .wplay file, or what the lesson is about]
---

Open Wordplay's creator for the author, at once, before anything else.
What they gave, if anything: $ARGUMENTS

1. Find the lesson. If they named a `.wplay` file, it is that one. If they
   named none and this folder holds exactly one `.wplay` file, it is that
   one. Otherwise start a new one: a file in this folder named for what
   they said the lesson is about, in lowercase words joined by hyphens
   (`two-clocks.wplay`), or `untitled.wplay` when they said nothing (with
   `-2`, `-3` after it if that name is taken), holding exactly:

   ```
   ---
   wordplay: 1
   ---

   :::tutor
   ```

   Never overwrite a file that is already there.
2. Call `preview_lesson` with its path. If it cannot open it, say why in a
   sentence and stop.
3. Tell the author, in two or three sentences, and nothing more: the
   creator is open in their browser (or, if the tool says it could not
   open one, the address to open); they write the lesson there, with its
   assistant and the tutor running on their own Claude, and it is saved to
   the file as they type; and this terminal has to stay open while they
   work, since the creator runs from it.

Do not write the lesson here unless they ask: the creator's own assistant
does that, in the page.
