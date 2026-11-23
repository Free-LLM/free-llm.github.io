---
title: "A dumb model, on purpose"
layout: default
parent: Blog
nav_order: 0.4
permalink: /blog/2026-11-15-a-dumb-model-on-purpose
---

# A dumb model, on purpose

_November 15, 2026_

*Why we are training a deliberately small Italian model with DCNR, how the first attempt stalled, and the three bugs that a copy task found for us.*

We are training a language model called `dumb`. The name is honest. It is a small Italian model that should, one day, hold a little conversation and do some basic reasoning. Nothing more.

The point is not the model. The point is the thing that trains it: [DCNR](/distributed-architecture), our runtime that splits a neural network into independent nodes and spreads them over ordinary machines. If DCNR cannot train a small model well, there is no reason to trust it with a big one.

This diary tells how it goes. Failures included, because they are the interesting part.

## The rules we gave ourselves

- **DCNR only.** No other training framework, not even as a reference to check against.
- **If a tool is missing, we build it** into DCNR, with tests, instead of working around it by hand.
- **Grow, do not restart.** Train a small network, then add layers and attention heads to it. The old nodes keep their weights and their "age"; the new ones start at age 0.
- **Fast loops first.** An experiment that does not fit in one idle window of the laptop cannot answer a question.
- **Every experiment says in advance** what it expects and what decision it informs.

One more rule: all of this runs on one laptop, and only in its idle hours. Training starts when the human says the machine is free, and stops when work resumes. A suspended run is normal, not a failure.

A note on who does what: an AI coding assistant runs the experiments and keeps the log; a human approves each stage and each idle window. The assistant once started a test run without asking whether the laptop was idle. It was reminded, stopped it, and wrote that in the log too.

## The first attempt: big, slow, stuck

Before this plan there was a simpler one: build the full model (8 layers, width 512, 8 attention heads per layer, a vocabulary of 32,768 tokens) and train it from scratch on a private library of Italian books.

It did not go well. We measure the loss as cross-entropy (CE) in nats: lower is better, and the perplexity is its exponential. After thousands of steps the model sat at a CE of about 4.70, a perplexity around 110.

On a fixed set of 40 Italian prompts it scored 0 of 20 acceptable continuations, and 19 of 20 fell into a loop. We gave it 1,000 more updates. They took 28 hours and 8 minutes, about 100 seconds per step. After the first 100, the remaining 900 updates improved the loss by 0.0073. Then 20 of 20 looped.

Along the way we found a small museum of mistakes:

- The learning-rate warmup was set to 26,873 steps, on a run planned for 13,446. With the clock it was using, the run would never have left warmup, and the learning rate was about 9 times lower than intended.
- The old training labels forced an "end of text" token at the end of every row of 512 tokens. The model learned it very well: on the row we probed, at position 512 it predicted the end with probability 0.99995.
- Layer normalization was initialized wrongly: its scale started near ±0.02 instead of 1. We first wrote that this was "a likely major cause" of the plateau. Then we ran a control on a small network with the old and the new initialization: 1.361 against 1.379. No real difference. We took the sentence back.

So we declared the plateau, froze that checkpoint as a reference, and wrote the plan above. Start with 2 layers and 4 heads. Make each step fast. Prove every piece.

## Known answers first

How do you know a training stack works? Give it a problem whose best possible loss you know.

We built two: a **copy task** (the second half of a sequence repeats the first; a model with working attention reaches a loss of 0) and **Markov chains** (random text with a loss floor you can compute).

Inside a test harness, without the distributed services, the small network passed: the copy task reached 0.0000 from update 85, and every Markov case ended within 0.05 nats of its optimum.

We also measured what a model with no intelligence gets on the library: 7.69 for word frequencies alone (unigram), 5.58 for a model that only looks at the previous token (bigram). A real model must beat 5.58.

## Bug one: the attention that was not there

Then we trained the 2-layer network for real, through the DCNR services: 6,000 updates, 98 million tokens.

It ended at 5.5859. The bigram baseline is 5.5816. After 6,000 updates, our transformer had tied with a model that looks at one token.

That is suspicious in a precise way: it is what you get if attention does nothing. So we ran the copy task through the services. It stayed at 4.16, which is ln 64, the best you can do without looking back, for all 400 updates.

The cause: in a brand new network, the attention heads tried to load their weights from storage. There was nothing to load. The services reported the missing object as an error, the code skipped the head, and the query and key weights stayed at the zeros they were allocated with. With both at zero their gradients are zero too. Attention was a plain average of the past, forever.

The test harness never saw it, because its fake storage answers "nothing here" without an error.

After the fix, the same copy task through the services: 4.16 at update 100, then 0.0347 at update 140 and 0.0164 at 300.

Every network we had trained from scratch on the services until then, including a whole learning-rate comparison, had no working attention. We threw them away and repeated the comparison.

## Bug two: GELU says NaN

The first learning-rate comparison had another surprise. Two of its runs started producing non-finite gradients; 72 updates were skipped, the script watching them took the silence for a stall and suspended them.

The cause was in the GPU code of the GELU activation: for any input above about 11 it returned NaN (a fast `tanh` overflowing). GELU(12) = NaN. A higher learning rate only made such values appear sooner. The fix clamps the argument; there is a test for it now.

## Bug three: the old AMD GPU

DCNR is meant for ordinary machines, so we added a second one: an old Intel MacBook with an AMD GPU.

Its first check gave a strange picture. In every batch of 32 sequences, the first one was right. The other 31 had a loss of exactly 10.397208, which is ln 32768: the model was answering "any token, equally" for them.

A batched matrix multiply shared one matrix across the batch. Apple GPUs broadcast it. The AMD driver computed only the first entry. After the fix the old machine trains correctly (the copy task ends at 0.0170, against 0.0164 on the laptop), just about 15 times slower: 55 seconds per update against 3.8.

We also split one small network across two Intel machines, with part of it on each. It solved the copy task too (0.0149), at about two and a half minutes per update. Slow, but it is the thing DCNR exists for.

## And then it learned

With attention working, the best peak learning rate changed (1e-3 instead of 3e-4), and the 2-layer network was trained again for 6,000 updates.

| Model | Validation CE |
|---|---|
| Unigram baseline | 7.69 |
| Bigram baseline | 5.58 |
| 2 layers, attention stuck at zero | 5.586 |
| 2 layers, attention working | 5.187 |

5.187 is 0.39 nats below the bigram baseline. A small step, but the first one we could trust.

One more thing happened in those days: profiling showed where the time went, and the update for this network went from 6.52 to 1.88 seconds in the test harness. Half of it had been a single operation.

Next: growing this network, layer by layer.

*The numbers here come from our training log. Runs use one random seed each, and the prompt suite was scored by the assistant, not by blind raters.*
