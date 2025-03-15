---
title: "Documentation is a Software Problem"
date: 2025-03-15T12:00:00-04:00
tags: []
featured_image: ""
description: "Rethinking how engineers can overcome writer's block."
---

Rethinking how engineers can overcome writer's block.

----
<!--more-->

These days I serve as a software architect for two open source projects -
[Shipwright](https://shipwright.io) and [Konflux](https://konflux-ci.dev). I tend to write more
prose than code, whether that be in the form of design documents or detailed feature requests.
If I'm lucky, I'll tackle a thorny bug or draft a proof of concept.

Both of these projects are in the business of building container images. Their documentation (as of
2025-03-14) also needs a lot of help.

## Docs and Code Aren't Different

Technical documentation and code are two sides of the same coin - both are means to express ideas.
Before I started writing software professionally, I thought code was just a way for humans to talk
to machines. My functions and "if-then-else" blocks were merely ways to translate my logic to the
1's and 0's of binary language.

This notion changed about 6 months into my first job, when a group of senior engineers at my
consulting firm organized a weekly "lunch and learn" seminar. The topic was the canonical "Gang of
Four" _Design Patterns_ [book](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8),
and every week we would read and discuss a different chapter. As we worked through each pattern, it
dawned on me that writing code also communicated ideas to my peers. Design patterns was one way to
organize ideas. Code comments, commit messages, and code review sessions also communicated my ideas
to my colleagues.

Documentation is likewise a way - the _first_ way - to communicate with our peers and the broader
world. If we as software engineers are in the business of communication, why is this part of the
job so hard?

## Why Do We Struggle Writing Docs?

First and foremost, there's the elephant in the room - the English language. As an industry we're
commanded to write in English first, yet for many software engineers English is not their primary
language. My youngest son is currently in 1st grade, and much of his early years have been spent
learning the rules and the many, many exceptions therein. English is a terrible chaos forged by an
astounding amount of bloodshed; no wonder many of my non-English peers lack confidence writing
docs.

Second, there is no compiler for good technical documentation, or rigid syntax checker that tells
us we are failing. [Vale](https://vale.sh/) comes close to this, but I have only encountered it in
a few projects. Otherwise we are left with spelling and grammar checkers, which I have never seen
block code from merging.

Lastly, I think we struggle to write documentation because we always treat it as a separate thing
unto itself. We don't think of docs as part of the software design process or development lifecycle
until the very end, if ever.

## Documentation as a Software Design Problem

What if we approached writing documentation in the same way we approached writing software
_as a product_? The questions we ask of good software products and docs are not that different:

- What is the problem we are trying to solve? And for whom?
- What does our ideal end state look like?
- What tools are at our disposal?
- How should we organize our solution?

I think if we as engineers design documentation in the same way we design software, we can get
out of the trap that leads us to throw docs "over the wall" or avoid writing them completely. In
the coming weeks I hope to explore these ideas deeper - stay tuned!
