---
title: "Back to the library (in progress)"
layout: default
parent: Blog
nav_order: 0.1
permalink: /blog/2026-11-23-back-to-the-library
---

# Back to the library (in progress)

_November 23, 2026_

*We mix the adult library back into the training of the stories model. The first mix made it forget the stories, the second fixed that, and then the run died on a batch made only of padding. Not finished yet.*

This post is about an experiment that is still running as we write. No conclusion today, only the story so far.

## The question

After [the stories ladder](/blog/2026-11-22-simple-stories) we have a model that writes simple Italian. Can it now learn the adult Italian of the library without losing the stories? And will it then pass the 40-prompt test that the library-only model [failed](/blog/2026-11-20-growing-on-a-library)?

The starting point, measured:

| Model | Loss on stories | Loss on the library |
|---|---|---|
| Stories model (S3) | 1.8885 | 6.3049 |
| Library-only model, same shape (G3b) | not measured | 4.4546 |

On library text the stories model does worse than a model that only looks at the previous token (5.58). It starts far away.

## The plan and its rules

Continue the same network, no growth, for 8,000 updates on a mix: 75% library batches, 25% story batches. The stories in the mix are the rehearsal that should keep the old skill alive.

Three things were fixed before starting:

1. **A forgetting guard.** If the loss on stories rises above 2.04, stop and report. The fallback would be a 50/50 mix.
2. **A progress check.** After 4,000 updates the library loss must be below 5.0.
3. **Predictions.** Library loss at the end: 4.6 to 5.0. Stories loss: 1.89 to 1.98.

## The 75/25 mix: the guard trips

| Updates | 0 | 500 | 1,000 | 1,500 | 2,000 |
|---|---|---|---|---|---|
| Library loss | 6.3049 | 5.2819 | 5.0334 | 4.8785 | 4.8413 |
| Stories loss | 1.8885 | | | | 2.1077 |

The library side went very well. In 2,000 updates it was already inside the range predicted for the *end* of the stage.

The stories side did not. 2.1077 is above the limit of 2.04, at the very first checkpoint. Our prediction for stories was simply wrong: one story batch in four did not prevent forgetting.

The guard did its job. The run was stopped at 2,000 of 8,000 updates.

## The 50/50 mix: the stories come back

The switch to 50/50 was the fallback named in advance. Its details were not: which library batches to use, the learning-rate schedule, the new guard (stories must not get any worse than 2.1077). Those were the assistant's choices, made after seeing the result above, and the log marks them as such.

One thing we did not test: whether simply lowering the learning rate would have saved the 75/25 mix. So we cannot say that the mix ratio alone was the problem.

Counting updates from the start of the stage:

| Updates | 2,000 | 4,000 | 6,000 | 7,000 |
|---|---|---|---|---|
| Library loss | 4.8413 | 4.6659 | 4.5850 | 4.5318 |
| Stories loss | 2.1077 | 1.9939 | 1.9469 | |

Both go the right way together. The stories loss came back below the original 2.04 limit, and the library loss kept falling: from 6.30 at the start to 4.53. That is already better than we predicted for the end of the stage, though not yet at the 4.4546 of the library-only model.

## The run that died on nothing

At 06:13 on the morning of November 23, at update 5,035 of the 50/50 part, the session failed with this message:

> batch has no supervised targets

The story batches had been built 128 rows at a time. The last one had a single real row; the rest was padding. Seen through the 32-row view used for training, that became three batches made of nothing but padding. The shuffle had put one of them at position 5,035.

A training session fails on such a batch instead of skipping it. The 75/25 dataset had the same three batches; the first run just never reached one.

Then the save after the failure timed out twice on one upload, over a slow connection, and succeeded at the third try. The weights in the running process were never lost.

The workaround for today: skip that position and train the remaining 965 updates as a separate last part. The proper fix (skip such batches, or never create them) is on the list, not built yet.

## Where we are

As of the morning of November 23: the last 965 updates are training. The stage total stays 8,000 updates, 2,000 on the 75/25 mix and 6,000 on the 50/50 mix.

**This is not finished.** Still to come:

- the final losses on the library and on the stories;
- the 40-prompt test, the real gate (16 of 20, fewer than 4 looping);
- the story test again, to see whether the stories survived.

We do not know how the gate will go. Before the run we wrote "uncertain", and we still think so. A lower loss did not move the story test last time.

The usual caveats apply: one run, one seed, no control experiment, and tests scored by the assistant rather than by blind raters, with the human having the last word.

We will write the result here, whichever way it goes.
