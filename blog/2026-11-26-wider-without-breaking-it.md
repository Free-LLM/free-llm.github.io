---
title: "Wider, without breaking it"
layout: default
parent: Blog
nav_order: 0.06
permalink: /blog/2026-11-26-wider-without-breaking-it
---

# Wider, without breaking it

_November 26, 2026_

*The model is wider now. It knows everything it knew before, on the first try minus one. Halfway through training it has not yet beaten its old self.*

[Last time](/blog/2026-11-24-more-training-same-stories) the rule we had written down said: more training at this size does not buy stories that hold together, so make the model wider. The tool to do that did not exist. Now it does, and it has been used once for real.

## What "wider" means

Every word, inside the model, travels as a list of numbers. Until now that list was 512 numbers long. Now it is 768. Each layer gets four more attention heads (12 instead of 8) and a bigger feed-forward part. The number of layers stays at 8.

The hard part is not making the model bigger. It is making it bigger **without making it forget**. The new numbers start at zero where they would write into the model, and small and random where they only read from it. The normalization layers, the ones that rescale everything at each step, at first ignore the new numbers completely, then let them in little by little over the first 1,000 updates.

The promise is precise: right after growing, the wider model must give exactly the same answers as the narrow one. We check it every time, on the same text, before training anything.

## First try: no

| | Loss before | Loss after growing |
|---|---|---|
| First try | 1.9039 | 2.2589 |

Not the same answers. The rule says: if this check fails, nothing is trained. So nothing was.

The cause was almost funny. To let the new numbers in "little by little", each normalization layer needs to know its own age: how many updates it has already done. We asked the system. The system answered 0 for all of them, because it only knows the age of the pieces it has in memory, and we had just restarted it. Age 0 meant "you have been letting the new numbers in since the beginning of time", so they were all fully in, from the first step. The check caught it. That is what the check is for.

The fix: load the model first, then ask the ages.

## Second try: yes

| | Loss before | Loss after growing |
|---|---|---|
| Second try | 1.9039 | 1.9039 |

The biggest difference, across all the measures, was 0.0000005. Same model, more room.

## Training, halfway

The plan is 6,000 updates on the same mix as the library stage: half simple stories, half the library. It runs only when the laptop is idle. This morning it stopped at 2,837, by request. Every 500 updates it pauses to measure:

| Updates | Loss on the library | Loss on the stories |
|---|---|---|
| Before growing | 4.5082 | 1.9039 |
| 500 | 4.6782 | 2.0637 |
| 1,000 | 4.6385 | 2.0277 |
| 1,500 | 4.6416 | 1.9968 |
| 2,000 | 4.5833 | 1.9687 |
| 2,500 | 4.5922 | 1.9728 |

Both got worse at first, as we expected while the new numbers were being let in. Since then they have been coming back down, slowly. Both are still above where the narrow model started.

We had predicted, for the end of the run, 4.20 to 4.40 on the library and 1.78 to 1.86 on the stories. Honestly, at this pace that looks optimistic. The learning rate is now going down, and in past runs a good part of the gain came in that last stretch, so it is not over. But we will not be surprised to have been wrong again.

## The cost

A wider model is slower. Each update takes about 18 to 20 seconds, against about 8 at the old width. The whole run is around 33 hours of machine time, spread over idle nights.

## What comes next

The remaining 3,163 updates, in the next idle windows. Then the same twenty story openings as always, and the prompts of the gate this model has never passed. The rule is written: 18 or more coherent stories out of 20 and width is helping; 16 or fewer and size is not the problem, so we go back to the data and the way we train.

The loss will tell us something before then. The stories will tell us what we actually want to know.
