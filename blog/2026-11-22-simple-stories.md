---
title: "Simple stories: the model starts to write"
layout: default
parent: Blog
nav_order: 0.2
permalink: /blog/2026-11-22-simple-stories
---

# Simple stories: the model starts to write

_November 22, 2026_

*A new ladder on short Italian children's stories takes the validation loss from 2.29 to 1.89. The model writes fluent little stories, and lands exactly on the edge of our test: 15 or 16 coherent out of 20, when 16 were needed.*

[Last time](/blog/2026-11-20-growing-on-a-library) the model had read a library of adult books and could produce one good clause before falling into a loop. The diagnosis was: too broad a diet for such a small model.

So we changed the diet.

## The corpus

We took `markod0925/TinyStories-Italian-Improved` from Hugging Face (Apache-2.0 on its card): a machine translation of TinyStories, short stories written with the vocabulary of a small child.

After filtering out empty rows, leftovers of markup, untranslated, duplicate, very short and very long stories, 704,295 of 705,882 remained.

| | Stories | Tokens | Batches (32 × 512) |
|---|---|---|---|
| Training | 697,359 | 145,359,219 | 8,873 |
| Validation | 6,936 | 1,448,535 | 89 |

The translation is not perfect. One of the first three samples we read had an English word left in it ("cercò some semi"). We kept it as it is.

A bigram model gets 3.918 on these stories. On the library it gets 5.582. That alone says how much simpler this text is. It also means that losses on the two corpora cannot be compared directly.

## The ladder, again

Same shapes and same recipe as the first ladder, from scratch, with no connection to the old networks.

| Stage | Shape (layers × heads) | Updates | Seconds per update | Validation CE |
|---|---|---|---|---|
| S0 | 2 × 4, from scratch | 3,000 | 4.4 | 2.2938 |
| S1 | 4 × 4, grown | 3,000 | 5.0 | 2.1256 |
| S2 | 4 × 8, grown | 2,873 | 5.7 | 2.0385 |
| S3 | 8 × 8, grown | 8,873 | 8.6 | 1.8885 |

S0 to S2 together read every story once. S3 read them all a second time, in a new order.

Before starting we wrote down predictions. For S0: between 2.6 and 3.2. It did better, 2.29. For S3: between 1.6 and 2.2. It landed inside.

All three growth steps passed the "same loss before and after" check. And after the second pass over the data the training loss (1.883) was level with the validation loss (1.8885): no sign of overfitting on the second pass.

One honest gap: there are no same-compute controls here. S1 is better than S0, but S0 trained for 3,000 more updates might have improved too. How much of each gain comes from the new layers and heads, and how much from simply training longer, is not measured.

## The story test

The old 40-prompt test is written in adult Italian, so for this ladder we made a new one: the openings of 20 validation stories, which the model never saw in training, frozen before the first update. The model continues each one for 64 tokens, taking the most likely token each time. The bar is the same: at least 16 of 20 grammatical and coherent, fewer than 4 looping.

"Coherent" has a rule: a continuation fails when it contradicts the opening or itself, or loses track of who is who. Drifting to another topic is allowed.

On S2: 20 of 20 grammatical. Coherent: 15. Or 16, depending on one story where two cats from the opening are simply dropped and a talking dog shows up. Two looping.

On S3: 20 of 20 grammatical. Coherent: 15. Or 16, depending on one story with "Jimmy e Jimmy". One looping.

Exactly on the line, twice. The log records both as "borderline", not as a pass. The project owner judged the gate passed both times; the assistant's count stayed at "15 or 16".

There was also a process slip: S3 was launched before the scoring of S2 was finished. The log says so. S3 could have been stopped at any time, but the order was wrong.

## What it writes

An opening about a girl and her dolls:

> A Lily piaceva giocare con le sue bambole. Ne aveva molte, ma la sua preferita era Pinki.

(*Lily liked to play with her dolls. She had many, but her favourite was Pinki.*)

S2 continued like this:

> Pinki era un gatto grande e marrone che amava correre e saltare. Un giorno, Lily e Pinki andarono al parco con le loro mamme.

(*Pinki was a big brown cat who loved to run and jump. One day, Lily and Pinki went to the park with their mums.*)

Good Italian. But Pinki was a doll. Failed.

S3, same opening:

> Era rosa e aveva i fiori sopra. A Lily piaceva molto Pinki. Un giorno, Lily e Pinki andarono al parco con la mamma. Lily vide un grande albero con tante foglie. Voleva arrampicarsi sull'albero. Chiese alla mamma:"Posso arrampicarmi sull'albero, mamma?"

(*She was pink and had flowers on her. Lily liked Pinki very much. One day, Lily and Pinki went to the park with mum. Lily saw a big tree with many leaves. She wanted to climb the tree. She asked her mum: "Can I climb the tree, mum?"*)

That one passes. And here is one that S3 fails, about two children at home with their father:

> Ma un giorno, il papà di Benny disse che dovevano andare a casa. Disse che dovevano andare a casa. Benny e Lilly non volevano andare. Volevano restare a casa. Dissero che non volevano andare. Dissero che erano tristi. Dissero che non volevano andare a casa. Dissero

(*But one day, Benny's dad said they had to go home. He said they had to go home. Benny and Lilly did not want to go. They wanted to stay home. They said they did not want to go. They said they were sad. They said they did not want to go home. They said*)

Told to go home while at home, and then the loop.

The pattern is clear. The grammar is there, the dialogue is there, the shape of a story is there. What gets lost is the specific content of the opening, replaced by stock scenes. "Un grande albero con un buco" (a big tree with a hole) appeared in four of the twenty S2 stories.

## What the test cannot tell

Between S2 and S3 the loss dropped from 2.039 to 1.889. The test did not move: 15 or 16, both times, on *different* stories. Five got better, four got worse.

With 20 items and one scorer, who is the assistant and not a blind rater, this test cannot tell S2 from S3. We write that down as a limit of the test, not as a result about the model.

## And on adult prompts?

We also ran the old 40-prompt suite on S3, as a diagnostic.

| | S3 (stories) | G3 (library) |
|---|---|---|
| Continuations flagged for repetition | 8 of 20 | 20 of 20 |
| Language items answered correctly | 4 of 10 | 0 of 10 |
| Answers read from the context | 1 of 10 | 0 of 10 |

Less looping, and some grammar items now come out right. But every prompt becomes a children's story. A recipe prompt continues with "la mamma di Lilia le diede un grande abbraccio" (*Lilia's mum gave her a big hug*).

That is what a model trained only on stories should do. It has learned to write. It has not yet read anything else.

Next: giving the library back to it, without letting it forget the stories.
