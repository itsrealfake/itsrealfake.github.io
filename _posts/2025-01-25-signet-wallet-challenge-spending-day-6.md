---
layout: post
title: 'Signet Wallet Challenge: Spending (day 6)'
date: 2025-01-25 11:29 -0600
---

# Day 6

not sure how much value i'm getting from the record of work, since i'm skipping days of work in between and not reading what i've already written. let's keep doing it anyway.

Another note to leave myself here: the dayjob wants me to work 2 consecutive days of the week, and they should be professional working days. so that's fine to get some cash, but it's going to impede my ability to get into this chaincode challenge on those days. Or at least, it will if i let it.

I'll take some time today to get some physical exercise in, as well. It's nice to be spending so much time learning a new language, but it's challenging on the body. My wrists and hands definitely notice the pain.

## execution

this `main.rs` file is looking for a `cookie` file in the bitcoin directory, so i'll point that at the actual location of the Bitcoin data directory, and perhaps that will help.

also, i'm aware that the main repo has diverged from the fork that i was provided, but it seems there's not a way for me to get those changes, as i can't see the main repo.

## moving on

our entry point to the spend/lib.rs is a `fn spend_p2wkph` like this: 

```
// Spend a p2wpkh utxo to a 2 of 2 multisig p2wsh and return the (txid, transaction) tupple
pub fn spend_p2wpkh(wallet_state: &WalletState) -> Result<([u8; 32], Vec<u8>), SpendError> {
    // FEE = 1000
    // AMT = 1000000
    // Choose an unspent coin worth more than 0.01 BTC

    // Create the input from the utxo
    // Reverse the txid hash so it's little-endian

    // Compute destination output script and output

    // Compute change output script and output

    // Get the message to sign

    // Fetch the private key we need to sign with

    // Sign!

    // Assemble

    // Reserialize without witness data and double-SHA256 to get the txid

    // For debugging you can use RPC `testmempoolaccept ["<final hex>"]` here

    // return txid, final-tx

    unimplemented!("implement the logic")
}
```

It seems like this first instruction, to choose an unspent coin worth more than 0.01 BTC should be our first order of operation. Since we are relying on the `struct WalletState` from before, let's see if we can find an unspent coin.

## time to do a little file cache

Since we're relying on the other program, we have to run 300 blocks worth of data before we can even start the spend script. instead, let's figure out how to cache this data to some type of file.

our struct is like this: 

```
// final wallet state struct
pub struct WalletState {
    utxos: Vec<Vec<u8>>,
    utxo_map: HashMap<(String, u32), UTXO>,
    witness_programs: Vec<Vec<u8>>,
    public_keys: Vec<Vec<u8>>,
    private_keys: Vec<Vec<u8>>,
}

struct UTXO {
    pub_key_or_witness: Vec<u8>,
    value_f: f64,
    value: u64,
}
```

i could output just some bytes, or i could output json. in either case, the program should look for the cache before calling the full program. and there might need to be some type of cache invalidation present. at first, i think some basic situation like:

```
fn get_wallet_from_cache(???) -> <WalletState, SomeKindOfError> {
  // look for a file that is the cache
  // if there is a cache 
        // if that cache is still valid
        // load the walletstate from cache
  // TODO: else 
        // get a wallet state
        // save that wallet state to the cache
  // return the wallet state
}
```

so, i'm going to start by sending this as a prompt to the LLM. and it did a fine job of answering the question. i'm left with another question:

### Thoughts about the approach here.

1. let's not break the functional code
2. let's not over complicate things...


## implement some file storage

i think it would be nice to let the `fn recover_wallet_state` accept a conditional parameter, but i'm not sure if rust does that easily (probably does?)

after digging in on this, it becomes clear that we'll need to serialize the object.

serializing the UTXO Map struct that i made turns out to be kinda painful, because tuples aren't natively supported serializable by serde or serde_json.

so the choice becomes: 

    a. rewrite the functional code to use the `Vec<Vec<u8>>` object type, potentially breaking our existing functionality (and remember there's only tests for some of our functionality).
    
    b. write `impl Serialize/Deserialize` for the UTXO Map struct. that'd be alright, but we don't actually know if that struct is going to be very helpful for us in the future.
    
it's a bit of a cognitive load to think about this, and that can be frustrating.

time to head for the gym.



