---
title: "Growing a model on a library: it works, and it is not enough"
layout: default
parent: Blog
nav_order: 0.3
permalink: /blog/2026-11-20-growing-on-a-library
---

# Growing a model on a library: it works, and it is not enough

_November 20, 2026_

*Adding layers and heads to a trained network preserves what it computes, and growth keeps up with training from scratch. But on a library of adult books the model stalls near perplexity 86 and fails the language test.*

In [the previous post](/blog/2026-11-15-a-dumb-model-on-purpose) we ended with a 2-layer network that finally learned something. This post is about making it grow, and about where that took us.

## How growth works

A transformer is a stack of blocks, and each block adds its output to a stream that passes through. So a new block whose last step starts at zero adds nothing: the network computes exactly the same function as before. The same trick works for a new attention head.

In DCNR each piece of the network is a separate node with its own age, and the learning rate depends on the age. After growth, the old nodes keep their age and go on learning slowly (at 1e-4). The new nodes start at age 0 and get the whole schedule: 300 warmup updates, a peak of 1e-3, then a slow descent.

Every growth step has a check: validate the network before and after on the same batches. The loss must be the same.

The first time, on a 2-layer network grown to 3: 6.051574 before, 6.051574 after.

## Is growing worth it?

A fair question: why not train the bigger network from scratch?

We ran the comparison once. The 2-layer network, grown to 3 layers and trained 4,000 more updates, reached 5.058. A 3-layer network trained from scratch for the same total time reached 5.013.

We had fixed a margin of 0.05 in advance, so this is a tie. To be fair to scratch: it is slightly ahead. To be fair to growth: scratch had about 18% more compute.

Then the grown network continued, and passed 5.013 after about 1,500 more updates. Getting a 3-layer model at that level cost 6.7 hours by growth (given the 2-layer one) against 11.4 hours from scratch.

One comparison, one seed. We kept growth as the default and did not repeat the control at the larger sizes.

## The ladder

| Stage | Shape (layers × heads) | How | Validation CE |
|---|---|---|---|
| G0 | 2 × 4 | from scratch, 6,000 updates | 5.187 |
| | 3 × 4 | +1 layer, 11,000 updates in all | 4.878 |
| G1 | 4 × 4 | +1 layer, 6,000 updates | 4.772 |
| G2 | 4 × 8 | +4 heads, 6,000 updates | 4.592 |
| G3 | 8 × 8 | +4 layers, 6,000 updates | 4.510 |

For reference: the bigram baseline is 5.58, and the old 8-layer model trained from scratch (same shape as G3) scores 4.739 on the same validation batches. G3 is 0.229 better.

## A bug found by a check that failed by 0.0017

The growth check before G2 failed. Not by much: 0.0017.

The growth was innocent. The grown network matched the *stored* copy of G1 exactly; it was G1 in memory that differed from G1 on disk.

The output layer scores the 32,768 tokens in tiles of 2,048. Once its bias had been updated on the GPU, every tile added the *first* tile's bias. Token number `j` was scored with the bias of token `j mod 2048`. A freshly loaded network did it right.

The effect was about 0.002 on the loss, and every run so far had it. Too small to change any conclusion, large enough to be caught by a check that expects zero. The next growth check, after the fix: 4.430472 on both sides.

## The language test: failed

Loss is not language. So there is a gate: 20 Italian prompts to continue, with the most likely token chosen at each step. To pass, at least 16 of 20 continuations must be grammatical and coherent, and fewer than 4 may fall into a loop. (Another 20 prompts ask for short answers and are reported separately.)

G3: **0 of 20**, and 20 of 20 looping. Short answers: 0 of 10 and 0 of 10.

The prompt:

> Quando Marta aprì la finestra, vide che durante la notte era caduta la neve. In strada non passava nessuno e

(*When Marta opened the window, she saw that snow had fallen during the night. Nobody was passing in the street and*)

G3 continues:

> non si vedeva più niente. La strada era deserta, e la gente si era fermata. La strada era deserta. La strada era deserta. La strada era deserta. La strada era deserta. […]

(*nothing could be seen any more. The street was deserted, and people had stopped. The street was deserted. The street was deserted...*)

And the phone one, where Roberto picks up the receiver:

> si mise a squillare.«Pronto?»«Sì.»«Pronto?»«Sì.»«Pronto?»«Pronto?»«Pronto?»«Pronto?» […]

(*it started ringing. "Hello?" "Yes." "Hello?" "Yes." "Hello?" "Hello?"...*)

Twelve of the twenty fell into the same little dialogue (*«Non lo so.» / «E allora?»*, "I don't know." / "So what?"), whatever the prompt.

There is progress in there: the first clause is often on topic, which the old model did not manage. And when we sample instead of taking the most likely token, the loops go away: 23 of 60 samples are mostly grammatical. But only 5 of 60 are also coherent.

## Was it the learning rate?

One suspicion remained: maybe the model was simply learning too slowly, with the old nodes at 1e-4.

So the assistant ran one more experiment on a copy of G3 (its own choice, while the next decision was open). First, 1,000 updates at a constant, higher rate of 3e-4. The loss got *worse*: 4.566 against 4.510, and the stop rule fired.

Then a second part, designed after seeing that result, which the log records as a departure from the rules fixed in advance: 2,000 updates descending from 3e-4 to 1e-4. Result: 4.4546. The estimate for G3 simply continuing at 1e-4 for the same number of updates was 4.465.

No difference. **The learning rate was not the limit.** On this corpus, at this size, the model gains about 0.015 to 0.02 nats per 1,000 updates. It sits at a perplexity of about 86.

## So what is the limit?

We looked at the data, without any model. The library is 4,626 books and 489.5 million tokens: adult literature of many kinds, much of it translated. Three things came out:

- About 1.3% is not Italian prose at all, and about 1% is split into single letters, probably by the text extraction. Worth cleaning, but far too little to explain the result.
- The vocabulary is uniform from book to book. There is no "simple" part to start from.
- It is broad: thousands of settings, registers and names. The model has about 42 million parameters and has seen each token about once.

Small models write coherent text when the language is restricted, or after far more reading than our idle hours allow. One more pass over the library at this size would cost 57 to 75 idle hours, for a gain that might be 0.2 to 0.3. Still short of the gate.

We also built weight decay into the optimizer, then skipped its comparison: little effect was expected while the model underfits.

So the decision: leave the library for now. Start a new ladder on something much simpler, and bring the library back later.

*Caveats, as in the log: one seed per run; the 16-of-20 gate is scored by the assistant, not by blind raters; the growth-versus-scratch control was run only at 3 layers.*
