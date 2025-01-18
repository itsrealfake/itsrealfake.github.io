---
layout: post
title: signet wallet challenge - day 2
date: 2025-01-17 12:20 -0600
---
 
# Signet Wallet Challenge - Day 2... 
 
(note: i'll probably group all these into a category or page at some later date, but this is mostly note-taking for me at this time)
 
## regrouping
 
yesterday, i started with base58_decoding of the private key for the wallet, where i want to recover funds.
 
the reason to do this, is because i need to eventually get the first bunch of addresses for this wallet, and then find out which of the UTXOs on the signet chain are opinted at my addresses. once i've done that, i should be able to count my sats and find out my balance.
 
so, decoding was like: create a bigint from the base58 `tprv...` then turn that bigint into a byte array. then chop off the checksum & return the byte-array
 
right now, i'm not confident that the decoding is done right. i also tried to calculate the checksum of the verification, in order to validate the bytes.

```
    // BONUS POINTS: Verify the checksum!
    if bytes.len() < 4 {
        return Err("Input too short to have a checksum");
    };
    let checksum_length = 4; // 4 bytes checksum in Base58 encoding
    let checksum = bytes.split_off(bytes.len() - checksum_length);
    
    // compute the touble SHA-256 hash of the data for the checksum (thats how bitcoin does it)
    let hash = {
        use sha2::{Sha256, Digest};
        let mut hasher = Sha256::new();
        hasher.update(&bytes);
        let first_hash = hasher.finalize_reset();
        hasher.update(&first_hash);
        hasher.finalize()[..checksum_length].to_vec()
    };
    
    // check if the computed checksum matches the provided checksum
    if checksum != hash {
        return Err("Checksum verification failed, bigly");
    };
```

now that i've reread the work from yesterday, this seems like it's doing the right thing. it's certainly possible that it'll turn out to be an issue later on. i'll have to wait and see.

one thought that occurs to me now is that i could write some tests for my code to see if it's doing what i need it to do. that's not a bad idea... but tests are a pain, so i can just skip them for now.

## next steps

The chaincode exercise includes a lot of `unimplemented!` functions which are going to get plugged into this public function, where we can see the next step is to "Derive the key and chaincode at the path in the descriptor (`84h/1h/0h/01`)":

```
// public function that will be called by `run` here as well as the spend program externally
pub fn recover_wallet_state(
    extended_private_key: &str,
) -> Result<WalletState, BalanceError> {
    // Deserialize the provided extended private key
    let deserialized_extended_private_key = base58_decode(extended_private_key);
    println!("deserialized_extended_private_key: {:?}", deserialized_extended_private_key);

    // Derive the key and chaincode at the path in the descriptor (`84h/1h/0h/0`)

    // Get the child key at the derivation path

    // Compute 2000 private keys from the child key path
    // For each private key, collect compressed public keys and witness programs
    let private_keys = vec![];
    let public_keys = vec![];
    let witness_programs = vec![];

    // Collect outgoing and spending txs from a block scan
    let mut outgoing_txs: Vec<Vec<u8>> = vec![];
    let mut spending_txs: Vec<Vec<u8>> = vec![];
    let mut utxos: Vec<Vec<u8>> = vec![];

    // Scan blocks 0 to 300 for transactions
    // Check every tx input (witness) for our own compressed public keys. These are coins we have spent.
    // Check every tx output for our own witness programs. These are coins we have received.
    // Keep track of outputs by their outpoint so we can check if it was spent later by an input
    // Collect outputs that have not been spent into a utxo set
    // Return Wallet State
    Ok(WalletState {
        utxos,
        public_keys,
        private_keys,
        witness_programs,
    })
}
```


### ugh... fine i'll write some tests

rust makes it easy enough to write tests... then you can run them easily.

`TODO: link to the article for making tests work`

so, let's try to do something like that for ourselves. where do i put this test code? it turns out that all it takes to add test is to include the `#[cft(test)]` & then a mod block afterwards, so i did that.

i took this opportunity to clean up the mess i made yesterday... and also to check my work. previously, i was chopping off bytes too many times. in determining how to write my tests, and what the expectations needed to be, i was able to correct my mistake.

```
#[cfg(test)]
mod tests {
    use super::*;
    use std::error::Error;

    #[test]
    fn test_base58_decode_correct_length() -> Result<(), Box<dyn Error>> {
        // This 78 byte structure can be encoded like other Bitcoin data in Base58
        // by first adding 32 checksum bits (derived from the double SHA-256 checksum)
        // and then converting to the Base58 representation. 
        // This results in a Base58-encoded string of up to 112 characters. Because of the choice 
        // of the version bytes
        // the Base58 representation will start with "xprv" or "xpub" on mainnet, "tprv" or "tpub" on testnet. 
        let expected_byte_len = 78;
        let result = base58_decode(EXTENDED_PRIVATE_KEY).unwrap().len();

        assert_eq!(result, expected_byte_len);
        Ok(())
    }

    #[test]
    fn test_base58_decode_too_short() -> Result<(), Box<dyn Error>> {
        let base58_encoded_string = "derp";
        let expected = Err(TOO_SHORT_FOR_CHECKSUM_ERROR);
        let result = base58_decode(base58_encoded_string);
        assert_eq!(result, expected);
        Ok(())
    }

    #[test]
    fn test_base58_decode_not_base_58() -> Result<(), Box<dyn Error>> {
        let not_base58_encoded_string = "0000";
        let result = base58_decode(not_base58_encoded_string);
        assert_eq!(result, Err(CHARACTER_NOT_BASE58_MESSAGE));
        Ok(())
    }

    #[test]
    fn test_base58_decode_fail_checksum() -> Result<(), Box<dyn Error>> {
        const EXTENDED_PRIVATE_KEY_BAD_CHECKSUM: &str = "tprv9999MBicQKsPdcpHEL3yNDcHEXKgJ1R8mPz1doXUyqAcRQh5Z7MxBj29hd8dnzTKSFW1DuCbDM8Co8zEKVtqToNWE6bzggXTS1vofwtFkRa";
        let result = base58_decode(EXTENDED_PRIVATE_KEY_BAD_CHECKSUM);
        assert_eq!(result, Err(CHECKSUM_VERIFICATION_FAIL_ERROR));
        Ok(())
    }
}
```

honestly, it's a big relief to have these tests in place. i'm using TDD at the day-job, and this will help me move more quickly, and confidently through the rest of this exercise. it was definitely worth a couple hours to get this working. (just checked... it was 128 minutes of effort & i wasn't rushing).

## Deserializing a Decoded `Vec<u8>`

our prompt looks like this:

```
struct ExKey {
    version: [u8; 4],
    depth: [u8; 1],
    finger_print: [u8; 4],
    child_number: [u8; 4],
    chaincode: [u8; 32],
    key: [u8; 32],
}

// Deserialize the extended pubkey bytes and return a ExKey object
// Bip32 Serialization format: https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#serialization-format
// 4 byte: version bytes (mainnet: 0x0488B21E public, 0x0488ADE4 private; testnet: 0x043587CF public, 0x04358394 private)
// 1 byte: depth: 0x00 for master nodes, 0x01 for level-1 derived keys, ....
// 4 bytes: the fingerprint of the parent's key (0x00000000 if master key)
// 4 bytes: child number. This is ser32(i) for i in xi = xpar/i, with xi the key being serialized. (0x00000000 if master key)
// 32 bytes: the chain code
// 33 bytes: the public key or private key data (serP(K) for public keys, 0x00 || ser256(k) for private keys)
fn deserialize_key(bytes: &[u8]) -> ExKey {
    unimplemented!("implement the logic")
}
```

### now i have a question!

we wrote our test to make sure that `base58_encode` would return `78 bytes`. and the spec from [BIP-32]() tells us that's correct. but our `struct ExKey` when added up is only 77 total bytes. this implies something is going to be lost. my first expectation is that the key returned is a bit-flipped public or private, where 0 is public and 1 is private. but that may not end up being the case... as the BIP specification tells us that the 4 version bytes should give us this info.

what's up with that?

looking into the Discord group, i see that a lot of people are just changing the size of this `ExKey::key` to 33, so i think i'll do that also.

### Also, an error Enum is discovered

This isn't necessary for me to deal with immediately, but i found this:

```
#[derive(Debug)]
pub enum BalanceError {
    MissingCodeCantRun,
    // Add relevant error variants for various cases.
}
```

I could modify those tests to expect messages from here, and not use `const` declarations for the error messages. probably a good practice to clean up my code, but i want to get moving sooner than later, so i'll let this slide for a while.

### Deserializing looks easy?

i'm not sure what the right syntax is to use, but i should be able to just `.split_off` bytes from the output of the decoder & return a struct that way. i'll have to remember how to do that.

okay.. well, it's not ultra easy, but it's not as disorienting since i've been looking at some Rust in the last day. first up, we have this `ExKey` struct which has the `key: [u8; 32]` value. my first thought was to make it a `33` but looking at [Learn Me A Bitcoin](https://learnmeabitcoin.com/technical/keys/hd-wallets/extended-keys/), i learn that it's a 32 or 33 value.

Private Keys are 32, Public keys are 33.

Also, we're passing this `tprv` by reference, and i wonder about how that's going to impact our memory usage. I'll bet it's gonna be a big deal.

Since we're only looking to collect our balance at this point in the challenge, i wonder if it would be okay to presume a public key is being used. Later, we can extend the `ExKey` to include private keys? Maybe that's too silly.

(( took a little break for dinner ))

Looking into the assignment a bit further, it's apparent that i can get away without playing around with the public/private key length difference. it's good to know that it's there though.

I think i need to look up some more information about wallet recovery steps. Now that i have the key

### Refactor the Base58 Decoder to use this new `BalanceError` enum

feels good to get that done, but i didn't really want to do it immediately. however, Rust kinda forces your hand with the compiler parts.

## But first, some downtime.

now i've got to get to bed, or at least go away from this computer screen. The next step should be implementing the secp256k1 i guess?

I'll read some of Mastering Bitcoin 3rd Edition, and perhaps things will start to become more clear to me. I'm definitely having trouble keeping up, and feel like the pace of the group is faster than my own pace. That said, i feel more comfortable in Rust the more time i spend with it.

certainly though, the challenge of thinking in bytes is not something I (a programmer who's used interpreted programs for all of my career) naturally understand.



