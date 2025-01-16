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
