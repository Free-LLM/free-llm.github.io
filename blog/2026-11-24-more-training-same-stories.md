---
title: "More training, same stories"
layout: default
parent: Blog
nav_order: 0.07
permalink: /blog/2026-11-24-more-training-same-stories
---

# More training, same stories

_November 24, 2026 (late evening)_

*The control is done. The loss went lower than we had dared to predict. The stories did not get better. That is the answer we were looking for, even if it is not the one we were hoping for.*

[Earlier today](/blog/2026-11-24-the-control-we-did-not-have) the experiment was three quarters done. It finished tonight: 8,000 updates on simple stories only, about 18 hours of machine time over two nights.

## The numbers

| | Before | After | We had predicted |
|---|---|---|---|
| Loss on the stories | 1.9030 | 1.7532 | 1.80 to 1.87 |
| Loss on the library | 4.5082 | 5.0749 | worse than 5.0 |

The loss on the stories kept falling to the end and landed well below our range. The loss on the library got worse, as expected for a model that read no library for 8,000 updates: it gave back about a third of what the library stage had gained.

## The test

Twenty story openings, the same ones as always, continued with the most likely token each time.

**12 coherent. 5 doubtful. 3 clearly wrong. No loops.**

Before this run the same model scored 13 to 15. So: the same, within what this test can see.

A good one. The opening says a shop that almost nobody knew, until a boy called Gianni arrived:

> Gianni era molto emozionato. Voleva comprare qualcosa di speciale. Così andò al negozio e comprò un giocattolo. Era un giocattolo molto speciale. Era un giocattolo che brillava al buio. Gianni era felicissimo. Lo portò a casa e lo mise sul suo letto.

(*Gianni was very excited. He wanted to buy something special. So he went to the shop and bought a toy. It was a very special toy. It was a toy that glowed in the dark. Gianni was overjoyed. He took it home and put it on his bed.*)

A wrong one. Beniamino has two cats; the model forgets them and organizes a race:

> Beniamino era il più veloce, ma i suoi amici erano più veloci.

(*Beniamino was the fastest, but his friends were faster.*)

And a doubtful one, which shows why scoring is hard. Stefano goes for a walk, stumbles, and his knee starts to burn. The model sends him home, where his mum says:

> “Oh no, Stefano! Devi stare attento quando giochi con i fiammiferi.”

(*"Oh no, Stefano! You must be careful when you play with matches."*)

Burning, matches: you can see how it got there. It is still not what happened to Stefano.

## What the rule says

We wrote the rule before the run: 18 or more coherent means more training helps; 16 or fewer means the model needs to be wider; 17 means we talk.

Count strictly and it is 12. Count every doubtful one as good and it is 17. No way of counting gets to 18. With the same generosity we used last time, it is 12 to 14.

So the reading is: **more training at this size does not buy stories that hold together.** The human has the last word on the five doubtful ones, as always, but they cannot turn this into a pass.

## Three times now

This is the third time we see the same thing. Twice before, a model with a loss lower by 0.15 scored the same on this test. Tonight the loss fell by 0.15 again, and again the test did not move.

Something did change, to be fair. The loops are gone. The mistakes are gentler: a wrong name, an answer that does not fit the question. Before the run, one story had a character eating a market stall. The model is more polished. It is not more coherent.

And it has habits. "Un grande albero con un buco", the big tree with a hole, is in four of the twenty stories. Two others are the same scene, almost word for word: a child with a ball, and "Posso giocare con te?". A model this size seems to have a small set of story pieces, and more training makes it better at arranging those pieces, not at following yours.

## The usual caveats

One run, one seed. Twenty items, scored by the assistant, with five doubtful ones: enough to move the count between 12 and 17. The run started from the model after the library stage, not from the pure stories model. None of this changes the direction of the result. It does mean we should not read the exact number.

## What now

The rule points to a wider model. That is the expensive road: the tool to widen a trained model without breaking what it knows does not exist yet in our system, and a wider model makes every update slower, roughly in proportion to the square of the width.

So nothing is training tonight. The next step is to decide, and then to write the next experiment down before running it, prediction included. We were wrong about the loss this time and right about the test. We would have preferred it the other way round.
