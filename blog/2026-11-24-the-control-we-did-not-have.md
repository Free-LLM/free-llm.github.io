---
title: "The control we did not have"
layout: default
parent: Blog
nav_order: 0.08
permalink: /blog/2026-11-24-the-control-we-did-not-have
---

# The control we did not have

_November 24, 2026_

*After the failed test, the honest question was: is the model too small, or just not trained enough? We are running the experiment that can tell. It is three quarters done, and one of our predictions is already wrong.*

[Yesterday](/blog/2026-11-23-library-stage-result) ended with a list of things we did not know. One of them was this: "If the same 8,000 updates had gone to stories only, would the story test be better? Not measured."

So we are measuring it.

## Why this, and not a bigger model

The model writes correct sentences and loses the thread after one or two of them. There are two simple explanations:

1. **It has not trained enough.** More updates at this size would fix it.
2. **It is too small.** More updates will lower the loss and change nothing you can read.

The two lead to different next steps. The first says: keep the size, train longer. The second says: grow the model wider, which is the more expensive road. Growing first and asking later would cost a lot and teach us little, so the cheap experiment goes first.

## The design

We took a copy of the model exactly as it came out of the library stage. The original is untouched. The copy gets 8,000 more updates on the simple stories only, no library at all, with the same learning-rate shape as before. Every 500 updates we measure the loss on stories it has never seen.

At the end, the copy sits the story test: 20 story openings to continue.

## The rule, written before the run

This is the part we care about most. The rule was fixed before the first update:

- **18 or more of 20 coherent:** training amount helps. Next step is longer training on a mixed corpus, before any growth.
- **16 or fewer:** more training at this size does not buy coherence. Next step is a wider model.
- **17:** too close to call. We report it and decide together.

For reference, the same test gave 13 to 15 just before this run, and 15 or 16 one stage earlier.

We also wrote down what we expected: a final loss on the stories between 1.80 and 1.87, a test score between 14 and 17, and a loss on the library worse than 5.0 (it was 4.51), because a model that reads only stories for 8,000 updates should forget some of the library.

## Where we are

The run started in the evening of the 23rd and went on through the night, about 8 seconds per update. In the morning the machine was needed for other work, so the run was paused at update 5,860 of 8,000, with its state saved. It picks up again this evening.

The loss on unseen stories so far:

| Updates | Loss on stories |
|---|---|
| start | 1.9030 |
| 500 | 1.9211 |
| 1,000 | 1.9020 |
| 1,500 | 1.8887 |
| 2,000 | 1.8737 |
| 2,500 | 1.8609 |
| 3,000 | 1.8490 |
| 3,500 | 1.8352 |
| 4,000 | 1.8236 |
| 4,500 | 1.8115 |
| 5,000 | 1.7994 |
| 5,500 | 1.7876 |

A small bump at the start, while the learning rate climbs, and then a very steady descent: about 0.012 every 500 updates, with no sign of slowing yet.

## One prediction is already wrong

We said the loss would end between 1.80 and 1.87. It passed 1.80 at update 5,000, with 3,000 still to go. We were too pessimistic: the model had more to learn from the stories than we thought.

That is good news for the loss. It is not yet news about anything else. We have been here before: two earlier stages differed by 0.15 in loss and scored the same on the story test. The whole point of this experiment is to see whether a lower loss, this time, turns into stories that hold together. The loss curve cannot answer that. Only the 20 continuations can.

## What this experiment cannot tell us

- The copy starts from the model *after* the library stage, not from the stories-only model before it. So this is "what does more story training do to this model", not a clean replay.
- One run, one seed.
- Twenty items, scored by the assistant. A result of 17 would mean very little, which is why the rule says so in advance.

## Next

Tonight the last 2,140 updates, then the two final measurements (stories and library), then the test. The next post has the score and the decision that follows from it, whichever way it goes.
