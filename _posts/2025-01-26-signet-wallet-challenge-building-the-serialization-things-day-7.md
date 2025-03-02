---
layout: post
title: 'Signet Wallet Challenge: Building the Serialization things (day 7)'
date: 2025-01-26 10:43 -0600
---

# Day 7

yesterday we tried making the file storage, we're still gonna be making that. in order to do so, we need a way to serialize / deserialize the struct WalletState. that struct also relies on the UTXO map struct, which is a hashmap, where the keys are outpoints.

outpoints are tuples `(txid, txout)` and that doesn't easily serialize by any native `serde` macros.

yesterday also included a good deal of refactoring of the 1000-line blob into several module files. (are they actually called module files? or is there some other correct terminology to use in Rust? perhaps the answer to this question will make itself known in the future).

## implement serialization for the Wallet State function

i was having trouble with the concept of defining a struct for a single property of another struct.

it ends up looking like this:

`pub struct UtxoMap(pub HashMap<(String, u32), UTXO>);`

```
pub struct WalletState {
    pub utxos: Vec<Vec<u8>>,
    pub utxo_map: UtxoMap,
    pub witness_programs: Vec<Vec<u8>>,
    pub public_keys: Vec<Vec<u8>>,
    pub private_keys: Vec<Vec<u8>>,
}
```

then we can create some helpers and de/serializers: 

```
impl UtxoMap {
    pub fn new() -> Self {
        UtxoMap(HashMap::new())
    }

    pub fn add_utxo(&mut self, txid: &str, vout: u32, utxo: UTXO) {
        self.0.insert((txid.to_string(), vout), utxo);
    }

    pub fn get_utxo(&self, txid: &str, vout: u32) -> Option<&UTXO> {
        self.0.get(&(txid.to_string(), vout))
    }
}
impl Serialize for UtxoMap {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: serde::Serializer,
    {
        use serde::ser::SerializeMap;
        let mut map = serializer.serialize_map(Some(self.0.len()))?;
        for ((txid, vout), utxo) in &self.0 {
            let key = format!("{}-{}", txid, vout);
            map.serialize_entry(&key, utxo)?;
        }
        map.end()
    }
}
```

there's a bit more than that, but this is the major bits.

## implement deserialization, from file cache to walletstate objects

okay, so now we need to get our wallet state from the file cache if it exists.


okay, this seems to be working correctly. we now have a cached wallet state, that can be used in the spending functionality (we hope).

## Given a wallet state, let's find an unspent transaction with a value greater than .01 btc

```
pub struct WalletState {
    pub utxos: Vec<Vec<u8>>,
    pub utxo_map: UtxoMap,
    pub witness_programs: Vec<Vec<u8>>,
    pub public_keys: Vec<Vec<u8>>,
    pub private_keys: Vec<Vec<u8>>,
}
pub struct UtxoMap(pub HashMap<(String, u32), UTXO>);
pub struct UTXO {
    pub pub_key_or_witness: Vec<u8>,
    pub value_f: f64,
    pub value: u64,
}
```

given a `WalletState`, find all the UTXOs that are more than 0.01btc in size.

let's first print their pubkey or witness to the console so that i can see what we're working with.

## create input from the utxo

We're asked to send a transaction with these values:

`FEE = 1000`
`AMT = 1000000`



### What are the steps to create a p2wpkh transaction?


1. Reverse the txid hash, so it's little-endian
2. Compute the destination output script and output 
3. Compute the change output script and output


## damn, i really thought this UtxoMap was a good idea, but it's become a real pain in the butt

in addition to not quite understanding the structure of a transaction, it's not easy to get the TXID in bytes, since i've turned it into a tuple.

maybe this can be resolved?


