---
title: "Wider, but not wiser"
layout: default
parent: Blog
nav_order: 0.05
permalink: /blog/2026-11-27-wider-but-not-wiser
---

# Wider, but not wiser

_November 27, 2026_

*The wider model finished its training and beat the narrow one on every number we measure. Then we read its stories, and they were about as good as before.*

[Last time](/blog/2026-11-26-wider-without-breaking-it) the model had just been made wider (768 numbers per word instead of 512) and was halfway through its training, still worse than the narrow model it came from. We said it looked optimistic. It got better.

## The second half

| Updates | Loss on the library | Loss on the stories |
|---|---|---|
| Before growing | 4.5082 | 1.9039 |
| 2,500 | 4.5922 | 1.9728 |
| 3,000 | 4.5102 | 1.9375 |
| 3,500 | 4.4768 | 1.9284 |
| 4,000 | 4.4529 | 1.9064 |
| 4,500 | 4.4297 | 1.9039 |
| 5,000 | 4.4055 | 1.8823 |
| 5,500 | 4.3806 | 1.8715 |
| **6,000** | **4.3830** | **1.8658** |

On the library it went below the old model at 3,500, on the stories only at 5,000. At the end: 4.38 on the library, inside the range we had predicted (4.20 to 4.40), and 1.87 on the stories, just above the range (1.78 to 1.86). We were wrong by a hair.

The whole run took about 29 hours of idle laptop time, over two nights.

## The stories

Then the test that matters: the same twenty story openings as always. The model continues each one, and we read the result. The rule written before the run: 18 or more coherent stories out of 20 means the extra width is helping; 16 or fewer means size is not the problem.

| | Narrow model | Wider model |
|---|---|---|
| Coherent stories, of 20 | 13 to 15 | **13 to 17** |
| Stories stuck repeating themselves | 1 | **0** |

The range is there because some stories are neither good nor bad. Thirteen are clearly fine; four have one odd turn. Three are clearly broken. In one, a doll called Pinki becomes "a big boy who loved playing with dolls". In another the shopkeeper says "you can't go in, it's dangerous", and so the child goes in.

All twenty are correct Italian, and the prose flows better than before. But no way of counting gets to 18.

We also ran the forty prompts of the gate this model has never passed: openings for adults, a few short questions. They went about as before. Still almost always a children's story, often a girl called Lilia, sometimes a phrase repeated until the end. One coherent continuation out of twenty (five being generous), and about as many right answers to the questions (6 of 20, against 5).

## What it means

A model that is better on every loss and the same in the stories. That is a real result, even if not the one we hoped for. The extra size went into predicting the next word a bit better, not into keeping a story together.

So, by the rule, the next step is not to make it bigger. It is to look again at what we feed it and how we train it. Which exactly is to be decided.

## On the side

Reading the stories took almost four hours: at this width the model takes more than a minute to write 64 tokens (a few dozen words), because for every new word it reads the whole text again from the start. That has to change, and it is next on the list, in parallel. A model that remembers what it has already read, within one conversation, will write much faster.
