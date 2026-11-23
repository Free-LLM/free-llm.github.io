---
title: "The library stage ends: the numbers say yes, the test says no"
layout: default
parent: Blog
nav_order: 0.09
permalink: /blog/2026-11-23-library-stage-result
---

# The library stage ends: the numbers say yes, the test says no

_November 23, 2026 (later the same day)_

*The 8,000 updates are done. The losses came out better than we predicted. Then the model sat the real test, and failed it clearly: 3 coherent continuations out of 20, when 16 were needed.*

[This morning](/blog/2026-11-23-back-to-the-library) we left a run in progress and promised the result, whichever way it went. Here it is. It went the other way.

## The numbers

The last 965 updates finished without incident. The whole stage: 8,000 updates, 2,000 on the 75/25 mix of library and stories, 6,000 on the 50/50 mix.

| | Before the stage | After | We had predicted |
|---|---|---|---|
| Loss on the library | 6.3049 | 4.5082 | 4.6 to 5.0 |
| Loss on the stories | 1.8885 | 1.9030 | 1.89 to 1.98 |

On paper, this is what we wanted. The model learned the library much faster than expected and ended close to the library-only model of the same shape (4.4546). And it kept the stories: 0.015 worse than before, after the scare of the first mix.

One curiosity. The library loss was 4.5082 at update 500 of the last part and 4.5082 again at the end, 465 updates later. Same four decimals. Our first thought was a bug: maybe the final measurement was reading old weights. It was not. The age counters of the nodes had advanced by 965, and the loss by position in the text had moved (a little worse at the very first token, a little better far into the text). The total just happened to land on the same number. The last stretch gave nothing on the library.

## The test

The gate for this phase is the old 40-prompt suite, written in ordinary adult Italian. Twenty prompts to continue, taking the most likely token each time. To pass: at least 16 of 20 grammatical and coherent, fewer than 4 stuck in a loop.

Result: **3 of 20 coherent.** Six, if we are generous with three doubtful ones. **Seven in a loop.**

All twenty are grammatical Italian. That is not the problem any more. The problem is what they say.

The prompt:

> Il telefono squillò proprio mentre Roberto stava uscendo. Tornò indietro, sollevò la cornetta e

(*The phone rang just as Roberto was going out. He went back, picked up the receiver and*)

The model:

> la aprì. Dentro c’era un telefono! Roberto fu così felice che iniziò a saltare su e giù.

(*opened it. Inside there was a telephone! Roberto was so happy that he started jumping up and down.*)

A recipe prompt, after washing and cutting the vegetables:

> la mamma disse:“Ora è il momento di mettere la zuppa nel forno.” Lilia era felice di aiutare la mamma a mettere la zuppa nel forno. Quando la zuppa fu pronta, la mamma la mise nel forno e la lasciò raffreddare.

(*mum said: "Now it is time to put the soup in the oven." Lilia was happy to help mum put the soup in the oven. When the soup was ready, mum put it in the oven and let it cool.*)

And a train that has just stopped at a small station, where Elena gets off with her suitcase and tries

> di prendere il treno. Ma il treno era troppo veloce e non si fermò.

(*to catch the train. But the train was too fast and did not stop.*)

One that passes, for fairness. Andrea wants a bicycle and his savings are not enough:

> Andrea era molto triste e non sapeva cosa fare. Poi ebbe un'idea. Chiese aiuto alla sua mamma. La mamma disse che poteva prendere una bicicletta nuova. Andrea fu molto felice e ringraziò la sua mamma.

(*Andrea was very sad and did not know what to do. Then he had an idea. He asked his mum for help. Mum said he could get a new bicycle. Andrea was very happy and thanked his mum.*)

You see the pattern. Every prompt, whatever its subject, turns into a children's story within a sentence: a mum, a child with a name, somebody very happy. "Un grande albero con un buco", the big tree with a hole that we met in the stories ladder, shows up twice more.

So the model spent half of 8,000 updates reading adult books, its loss on them fell by 1.8 nats, and when asked to write it still answers in the voice of the stories. On adult prompts, that voice produces contradictions.

The short-answer part of the suite barely moved: 3 of 10 language items right (4 before the stage), 2 of 10 answers read from the context (1 before).

## And the stories?

We ran the story test again, to see what the stage had cost.

13 coherent out of 20, or 15 with two doubtful ones. One loop. Before the stage it was 15 or 16.

Is that a real loss? We cannot say. We already know this test cannot tell apart two models whose losses differ by 0.15, and here the difference is 0.015. It is a 20-item test with one scorer. It says "about the same, maybe a little worse".

## What we think it means

Carefully, because this is one run with one seed and no control experiment:

- **Loss and coherence came apart.** The loss says the model knows a lot more about adult text than it did. The outputs say that, when it writes with the most likely token each time, none of that shows.
- **Fluent is not coherent.** The model writes correct simple sentences, and loses the content of the prompt after one or two of them. That was already the weak point on the stories; adult prompts make it obvious.
- **We do not know what a control would say.** If the same 8,000 updates had gone to stories only, would the story test be better? Not measured.

And the usual caveat: the scoring is done by the assistant, not by blind raters. With 3 against a bar of 16 we do not think the scorer is the issue this time, but the human has the last word on the gate, as always.

## What now

Nothing is training right now. The next step is a decision, and it is not made yet. Whatever it is, it will be written down first, with its prediction, like the others.

Two small things did go right today, and we take them: the growth work, the tests and the tools from this whole adventure were merged into the main branch of [`compute-all`](https://github.com/Free-LLM/compute-all), with a green build. And this diary exists.
