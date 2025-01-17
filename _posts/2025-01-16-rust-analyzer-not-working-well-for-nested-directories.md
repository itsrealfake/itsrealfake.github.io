---
layout: post
title: rust-analyzer not working well for nested directories
date: 2025-01-16 19:47 -0600
---

# Sometimes `rust-analyzer` wont start up in your project in VScode

you should add the right directories to make this work

at the project root, you should have a `Cargo.toml` file looking like this:

```
[workspace]
resolver = "2"
members = [
    "solution/rust/balance",
    "solution/rust/spend"
]
```

shoutout @shimmy6 for the heads up on this

