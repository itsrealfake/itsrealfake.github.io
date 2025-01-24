---
layout: post
title: Signet Wallet Challenge (day 4)
date: 2025-01-21 08:56 -0600
---

# Day 4

I've spent a couple days decompressing with friends. There was some snow, which is pretty to look at, but perhaps not the nicest thing to deal with if you're living outside. Fortunately, i'm not living outside any longer. Let's count our blessings.

While i was resting, i took some time to read up on Private and Public keys in Mastering Bitcoin 3rd Edition. In my reading, I was left with a question about computing child keys. It seemed to me that I needed to compute all ancestor keys before computing decendants, but the LLM has informed me this is incorrect.

In other words, it's possible to compute the Child Key at N without computing the Child Key at N-1. This is great, but i wasn't certain about it from my reading.

## Late in the night when the things get weird

So i've done that thing where i get into the code without taking the notes, and it's a bit weird.

i'm not sure what i've done... is this an anti-pattern?

### For sure it's an anti-pattern

However, it allows us to move quickly, and then we can come back and recover by writing some tests of the funcitons that weren't previously tested.



 
