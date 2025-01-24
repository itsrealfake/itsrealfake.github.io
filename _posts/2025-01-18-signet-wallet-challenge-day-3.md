---
layout: post
title: Signet Wallet Challenge (day 3)
date: 2025-01-18 11:39 -0600
---


# Day three (already?!?)

okay, so it's a saturday and we all know what that means. or do we? i'm not sure we all do.

last night i fell asleep around 20h00 reading Mastering Bitcoin (3rd Edition) Chapter 4. Wow, it sure would have been helpful to read those chapters 3 & 4 before getting into the RPC CHallenge that came before this. Likewise, some of the challenges of the past couple days could have been informed by having done the reading.

> Lesson Learned?: Do the reading.

# Deriving the key and chaincode at the path in the descriptor

**What is a seed?**

just an integer. 64 bytes. it gets HMAC'd

**What is a chain code?**

integer. 32 bytes. the last half of the result of hashing the seed.

**What is a key?**

integer. 32 bytes. the first half of hashing the seed.

## let's apply it.

Here's our setup from yesterday, and we're going to apply this ExKey object to our task today. Hopefully, we'll get some momentum.


```
// Deserialize the extended pubkey bytes and return a ExKey object
// Bip32 Serialization format: https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#serialization-format
// 4 byte: version bytes (mainnet: 0x0488B21E public, 0x0488ADE4 private; testnet: 0x043587CF public, 0x04358394 private)
// 1 byte: depth: 0x00 for master nodes, 0x01 for level-1 derived keys, ....
// 4 bytes: the fingerprint of the parent's key (0x00000000 if master key)
// 4 bytes: child number. This is ser32(i) for i in xi = xpar/i, with xi the key being serialized. (0x00000000 if master key)
// 32 bytes: the chain code
// 33 bytes: the public key or private key data (serP(K) for public keys, 0x00 || ser256(k) for private keys)
fn deserialize_key(bytes: &[u8]) -> Result<ExKey, BalanceError> {
    // presently will only work for private keys.
    if bytes.len() != 78 {
        return Err(BalanceError::Expect78BytesForExKey);
    }

    let exkey = ExKey {
        // currently only works for private keys
        version: slice_to_array(&bytes[0..4])?,
        depth: slice_to_array(&bytes[4..5])?,
        finger_print: slice_to_array(&bytes[5..9])?,
        child_number: slice_to_array(&bytes[9..13])?,
        chaincode: slice_to_array(&bytes[13..45])?,
        key: slice_to_array(&bytes[45..77])?, // This one will be harder :-/ needs to be updated to handle Public Keys
    };

    Ok(exkey)
}

```

#### leveraging the LLM.

i'm using Grok to get through the work, so i've given it some instructions:

```
Don't provide the answers, but understand that I'm trying to get to the implemented logic, as part of my learning.

I'm using Rust to accomplish this, but I'm not very familiar with rust yet. I've written code for quite a while, so I'm familiar with many of the concepts, but I wouldn't say I know them well yet.

when you're providing answers, it would be helpful if you can give idiomatic code, in small parts, and explain the steps in comments.

```

I've got a local LLM running on my M2, but it's a bit too slow for this sort of back and forth, so i'm working with Grok (since it's free to me).

### deriving the public key

so, i wound up getting a response from the grok that suggested i only needed to use the private key & not the chaincode. it didn't make sense why i would only use 32 bytes, instead of 64 (key + chaincode). so i asked "why?" (always a good idea when trying to learn, or so i've heard).

it's answer was that the chaincode is used in the HD Wallet process. but that a public key can be derived from the private key alone:

> public = private * G (where G is the Generator point)

the chaincode is used to ensure that child addresses are derived determinitstically. i'll figure that out later, i suppose.

i also asked about how the function would look if i were to just do the multiplication myself. i was quite surprised to see that it required so much code to multiply this key by a G. i'll leave that for later as well... i really want to get through this project as quickly as i reasonably can.

in the end, using the SEcp256k1 library is pretty straigtforward:

1. create a secp context
2. create a secret key from the key bytes
3. create a public key from the context and the secret
4. serialize the public
5. return the Vec of the public serialized (to match the function output)

### deriving the private child 

the function looks like this:

> `derive_priv_child(key: ExKey, child_num: u32) -> ExKey `

we'll use this function to generate 2000 private child keys (and public), so that `child_num` is going to be iterating, but the rest will stay the same.

part of me wants to optimize this ahead of time, and make a tool that returns several items.

it's tempting to prematurely optimize because i can see these instructions later in the wallet state recovery challenge:

```
    // Compute 2000 private keys from the child key path
    // For each private key, collect compressed public keys and witness programs
    let private_keys = vec![];
    let public_keys = vec![];
    let witness_programs = vec![];
```

#### TIL: in Bip 32 we don't have a way to get public child key from private child keys

in the case for 'non-hardened' keys, anyway... we can't. this is because it would be too easily compromised. instead, we use "hardened" children (hard times make hard children, i guess).

the child private key is derived using the parent's private key, the chaincode, and the index of the key...

according to the grok:

> `private_child = hmacSha512(parent_private || index) + parent_private`

the `||` denotes concatenation.

### deriving the public child

this requires the index of the child key, and then some of the details we picked up in `ExKey`.

> Given a parent extended key and an index i, it's possible to compute the corresponding child extended key. The algo depends on whether the child is hardened or not (i -gte 2<sup>31</sup>), and whether we're talking about priv or pub keys. [bip 32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#child-key-derivation-ckd-functions).



