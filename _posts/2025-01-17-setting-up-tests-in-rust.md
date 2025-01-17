---
layout: post
title: setting up tests in rust
date: 2025-01-17 12:41 -0600
---

Tests are run like `cargo test` and you just create a setup like this in your code somewhere... and it will work.

how do we do it tho?

```
#[cfg(test)]
mod test {
    use super::read_compact_size;
    use super::Error;
    #[test]
    fn test_read_compact_size() -> Result<(), Box<dyn Error>> {
        let mut bytes = [1_u8].as_slice();

        let count = read_compact_size(&mut bytes)?;
        assert_eq!(count, 1_u8 as u64);

        let mut bytes = [253_u8, 0, 1].as_slice();
        let count = read_compact_size(&mut bytes)?;
        assert_eq!(count, 256_u64);

        let mut bytes = [254_u8, 0, 0, 0, 1].as_slice();
        let count  = read_compact_size(&mut bytes)?;
        assert_eq!(count, 256_u64.pow(3));

        let mut bytes = [255_u8, 0, 0,0,0,0,0,0,1].as_slice();
        let count  = read_compact_size(&mut bytes)?;
        assert_eq!(count, 256_u64.pow(7));

        let hex = "fd204e";
        let decoded = hex::decode(hex)?;
        let mut bytes = decoded.as_slice();
        let count  = read_compact_size(&mut bytes)?;
        let expected_count = 20_000_u64;
        assert_eq!(count, expected_count);

        Ok(())
    }
}
```
