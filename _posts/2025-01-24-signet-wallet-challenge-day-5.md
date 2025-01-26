---
layout: post
title: Signet Wallet Challenge (day 5)
date: 2025-01-24 10:56 -0600
---

# Day 5

I got to the part of the challenge where it was time to scan the blockchain for incoming and spending transactions. This step takes a while, there's a good deal of recursion pulling a block, looping on all its transactions, looping on all the vouts and vins for one of those transactions, and comparing private and public keys for each of those.

My first results were not promising. The program was not finding any transactions on chain.

After investigating the code i'd put together iwth the LLM, it became clear that my program wasn't correctly deriving the `ExKey` objects. In order to resolve that, i wrote some tests of the functions that were performing the HMAC-SHA512 step.

## Now what?

At this point, I've taken the time to add my descriptor to bitcoin-cli wallet & i see that the application is reporting there's something like 25 bitcoin in my wallet, so i know that these blocks hold some bitcoin for me.

Additionally, i'm now able to find blocks with incoming and outbound transactions for addresses in my wallet.

The next step will be to accumulate the values of those UTXOs.

Oh wait... our program is stuck in some sort of circumstance.

We're hanging on blockheight 114, but why?

## Why is the program hanging?

In order to figure this out, we should probably setup some logging to see what's being repeated. (I'm assuming that it's repeating itself).

...

turns out it wasn't hanging. but there was a lot of duplicated calculation happening. i spent time learning more about what i'd done in each of the functions, and realized there was a lot that didn't need to be there.

after that, it sped up alot.

## Finished it

In the end, after getting the wallet details into the CLI, i was able to look at my output and validate some of the outputs.

using `serde_json` was fine, but the `Float` vs. `Unsighned` integer math was funky. Also, the display function in the script being run by the test was doing some rounding of its own.

I also finished up the whole balance program without following the recommendations of the prompt: keeping track of all the UTXOs in a `Vec<Vec<u8>>` which is probably going to bite me in the ass later.

We'll see how to deal with it. I'm still not sure what exactly a `UTXO` data object should look like, which is part of why i didn't know how to track all of them.

I think the recommendation is to have a UTXO be a TXID + Vout number. this is the "outpoint". I ended up keeping them in a hashmap, `(txid, out num): value` and that worked fine for the calculations part.

