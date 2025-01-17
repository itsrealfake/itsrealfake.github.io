---
layout: post
title: BOSS Signet Wallet Challenge
date: 2025-01-16 14:17 -0600
---

# BOSS Signet Wallet Challenge

## Week 1: Recover Wallet State

### Getting going with Rust!

I'm working on this task using Rust. Before i could get started i had to remember how to run a rust project.

`cargo run`

And as i get into this challenge i find that there's a step for me to follow, an error message `not implemented: implement the logic`. So i got nosey with the code, and found myself back in the README where i learned that "the easy way" is to "use python".

Before i could use python, i needed to remember how to run a python project.

### Getting going with Python

Well, it's never that easy to use these tools is it? I'm on a mac, and i don't remember how python or python3 or whatever it is was installed... but i do remember that a long time back, i tried to use `vanitygen` to create a fancy address for myself (against the advice that it's probably going to be poisoned by broken encryption). I don't remember ever actually accomplishing the vanity address, but i do remember that it used python.

Thank goodness for bash history!

`CTRL + r` `vanity` gets me into the part of my bash history where i was using those commands... and then some up-arrow down-arrow navigation got me back to where i remember something about initializing an environment from which python can run:

`python3 -m venv venv`

`chmod +x run_balance.sh `

```
./run_balance.sh: line 2: pip: command not found
./run_balance.sh: line 3: python: command not found
```


oh great! i forgot the simplest step, `source venv/bin/activate`

#### Oh look, we're going!

Now i'm able to run the script `./run_it.sh` but we're getting outputs that imply the python code is not just incomplete, but actually not correctly formatted, so i spent a bit of time plugging in return statements at the correct indentation. because that's about as fun as making a new blog instead of doing the code challenge.

alright, as you might have imagined (if you'd been me, thinking ahead) it's no use to do this thing in python since you're intention was to do it in rust. somebody somewhere said it's easier, but i'm not sure it actually is, given that i'm not much familiar with python anyway.

so, python's got its own set of challenges, and frustrating syntax issues. i don't want to work with it. and while the completing the python version of this code is supposed to be "the easiest way" i'm not sure that it actually will be. so i'll take a look at the challenge in 

let's see what it looks like to do it in rust!

### Getting started in Rust, again

```
*[main][~/code/boss/signet-wallet-project-itsrealfake]$ ./solution/run_balance.sh
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `solution/rust/target/debug/balance /Users/itsrealfake/.bitcoin/signet/.cookie`
thread 'main' panicked at balance/src/lib.rs:34:9:
not implemented: implement the logic
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
well, well, well... we're back to where we started, kinda!

it looks like there's a clue here from rust "run with `RUST_BACKTRACE=1`". that helps us get further, and we learn that we can pass `=full`, too. using the bigger backtrace yields too much information for now, so let's go with "1". we find that this method (is it called that in rust?) is being called in the `main.rs` file.

hooray, we've got an entry point.

#### Great googly moogly... there's a lot to discover about how to do this task

First things first, i need to learn how to convert a base58 string to a bit integer. my initial thought was that an LLM could help a lot. And to be sure, it probably would but i'm trying to keep my learning going well, and so if i'm going to be implementing the things for myself, i can at least cannibalize existing opensource software instead of leaning super hard onto the LLM.

there's some instructions for me to inspect in this code...

```
// convert base58 string to a big integer
// convert the integer to bytes
// chop off the 32 checksum bits and return
// BONUS: verify the checksum!
```
i only vaguely remember addressing much of this in the initial introduction to Rust. and it definitely didn't get this detailed. i'm going to have to do some reading.

let's see what the [docs](https://docs.rs/bs58/latest/bs58/) & [source code](https://github.com/Nullus157/bs58-rs?tab=readme-ov-file) have to say about the bs58 library & how that's done in there. The first thing to discover is that there's several libraries offering this functionality.

looking at the source there, and having so little Rust experience was pretty unfulfilling. i was able to decipher some of what's written, and something i noticed was that both the base58 libraries i looked at were relying on `FromStr`. now, i've gone back to the Chaincode discord, and looked at some of the comments from other participants, and i see they're offering the hint to use this library: `num-bigint`, but i don't see anybody talking about `FromStr`. Wonder why, but for now, i want to move forward. Let's get this helper library running:

```
// including a library in Rust, add a dependency to the cargo.toml file
[dependencies]
num-bigint = "0.4"

```

updating the .toml file and running the script again, i see that `cargo` is looking out to crates.io. so i guess that's called a crate, not a library? worth considering what the answer to that question is, but not now... moving on.

let's look at the docs for [num-bigint](https://docs.rs/num-bigint/latest/num_bigint/). hmm... thot's not particularly helpful, yet. there's no "from string to bigint" functionality included in that crate. but... i remember from looking at the python recommendations from the LLM that there's a sort of algo for decoding a base58 string into a big integer.

#### let's convert a base58 string to bytes

so let me restate that, roughly:

```
base58_alphabet = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz"
decoded_big_int  = 0
for char in base58_str:
    # find the index of the character in the base58 alphabet
    value = base58_alphabet.index(char)
    # multiply the index of the char * 58 & add it to your accumulator
    decoded_big_int = decoded_big_int * 58 + value
    
return value
```

so i took some time with the LLM, and the [Rust Playground](https://play.rust-lang.org/) to get to a kind of solution for what i'm first trying to do.

```
use num_bigint::BigInt;

fn main() {
    // base58 string
    let base58_str = "<<Base58 Encoded String>>";
    
    let base58_alphabet = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz";
    
    // convert string to byte array
    let bytes = base58_str.as_bytes();

    let mut result = BigInt::from(0);  // Assuming you have `BigInt` from `num-bigint`
    let base = BigInt::from(58);

    for &b in bytes {
        let index = base58_alphabet.find(char::from(b)).expect("Character not in Base58 alphabet");
        result = result * &base + BigInt::from(index);
    }

    println!("Decoded: {}", result);
}
```

i didn't know if my work was right, and there's a bonus point available for verifying the checksum. so i worked with the LLM some more and broke out the checksum verification. i've wrapped up the base58_decode step for the day. and i need to get some dinner, seems like i spent about 3 hours to get this far, and i was taking a lot of notes in this document.

