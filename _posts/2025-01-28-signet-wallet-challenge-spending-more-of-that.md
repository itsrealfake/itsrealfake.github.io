---
layout: post
title: 'Signet Wallet Challenge: Spending... more of that'
date: 2025-01-28 18:57 -0600
---

# Day whatever...

okay, this is taking too long. i took time to read the details of Transaction construction in Mastering Bitcoin.

now i understand that the most important part of the construction is finding the `txid` so, that's what i'm going to focus on for the moment.

something that's super important about bitcoin is this rule that a UTXO must be spent the whole way for it to be used at all... so there's no such thing as using a UTXO twice.

this was new info for me. i had previously sullied my understanding by confusing UTXOs and addresses. what hadn't really clicked for em is that `txid` is the main way to keep track of bitrcoin in the system.

the address is used as part of the data that gets hashed to create the `txid` but other than that, it's really just a secondary piece of information

another thing that i learned in reading was that the way to get permission to spend a UTXO has changed over time:

public key -> private sig (version 0)
output script -> input script
witness program -> witness structure

it's this last version of getting permission to spend that's the focus of this current exercise.

