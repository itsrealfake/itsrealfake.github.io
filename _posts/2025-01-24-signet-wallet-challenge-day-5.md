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

