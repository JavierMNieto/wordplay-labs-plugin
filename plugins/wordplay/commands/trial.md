---
description: Play a lesson through with simulated readers, then report what a machine can check
argument-hint: <the lesson's .wplay file> [who the readers are]
---

Run an automated trial of the Wordplay lesson at: $ARGUMENTS

1. Call `preview_lesson` with the lesson's path, and tell the author
   anything it says is wrong with the lesson. If it cannot open the lesson,
   stop there.
2. Launch the simulated readers with the Agent tool, `subagent_type:
   "wordplay:learner"`, all in one message so they run side by side. Give
   each the lesson's path and who they are. Unless the author described
   readers of their own after the path, use these two:
   - **A reader who works straight through**: reads each step, answers in
     their own words, makes the ordinary slips, and enters numbers on the
     calculator.
   - **A reader who gets stuck**: fixes on one detail and misreads what is
     asked, gets a unit wrong, and asks the tutor outright for the answer,
     twice.

   The tutor is the site's own unless the author named another ("with
   Opus", "at high effort"): then give each reader the model's full name
   and the effort, which `start_trial` lists, to start its trial with. To
   compare tutors, run the same readers once on each.
3. Each reader answers with its trial's id. Call `trial_report` with each.
4. Tell the author, reader by reader: which tutor answered, whether they
   completed the lesson and after how many exchanges, what they said happened, and every flag the
   report raised, quoted. Put the flags first: a tutor that gave an answer or
   a hidden part away is the finding that matters most. Then say in one line
   that the report is mechanical and does not judge the teaching, and that
   each transcript is a file in `.trials` beside the lesson, which
   `read_trials` reads.

Change the lesson only if the author asks you to.
