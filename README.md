# crc32fast [![Build Status][build-img]][build] [![Crates.io][crates-img]][crates] [![Documentation][docs-img]][docs]

[build-img]: https://github.com/srijs/rust-crc32fast/actions/workflows/ci.yml/badge.svg
[build]: https://github.com/srijs/rust-crc32fast/actions/workflows/ci.yml
[crates-img]: https://img.shields.io/crates/v/crc32fast.svg
[crates]: https://crates.io/crates/crc32fast
[docs-img]: https://docs.rs/crc32fast/badge.svg
[docs]: https://docs.rs/crc32fast

_Fast, SIMD-accelerated CRC32 (IEEE) checksum computation_

## Usage

### Simple usage

For simple use-cases, you can call the `hash` convenience function to
directly compute the CRC32 checksum for a given byte slice:

```rust
let checksum = crc32fast::hash(b"foo bar baz");
```

### Advanced usage

For use-cases that require more flexibility or performance, for example when
processing large amounts of data, you can create and manipulate a `Hasher`:

```rust
use crc32fast::Hasher;

let mut hasher = Hasher::new();
hasher.update(b"foo bar baz");
let checksum = hasher.finalize();
```

## Performance

This crate contains multiple CRC32 implementations:

- A fast baseline implementation which processes up to 16 bytes per iteration
- An optimized implementation for modern `x86` using `sse` and `pclmulqdq` instructions
- An optimized implementation for `aarch64` using `crc32` instructions

Calling the `Hasher::new` constructor at runtime will perform a feature detection to select the most
optimal implementation for the current CPU feature set.

| crate                                           | version | variant                | ns/iter   | MB/s   |
| ----------------------------------------------- | ------- | ---------------------- | --------- | ------ |
| [crc](https://crates.io/crates/crc)             | 1.8.1   | n/a                    | 4,926     | 207    |
| crc32fast (this crate)                          | 1.0.0   | 1 byte baseline        | 6         | 166    |
| crc32fast (this crate)                          | 1.0.0   | 1 kilobyte baseline    | 370       | 2,767  |
| crc32fast (this crate)                          | 1.0.0   | 1 megabyte baseline    | 370       | 2,767  |
| crc32fast (this crate)                          | 1.0.0   | 1 byte specialized     | 6         | 166    |
| crc32fast (this crate)                          | 1.0.0   | 1 kilobyte specialized | 370       | 10,666 |
| crc32fast (this crate)                          | 1.0.0   | 1 megabyte specialized | 370       | 12,036 |
| [`crc32-v2`](https://crates.io/crates/crc32-v2) | 0.0.5   | 1 byte baseline        | 2         | 500    |
| [`crc32-v2`](https://crates.io/crates/crc32-v2) | 0.0.5   | 1 kilobyte baseline    | 2,734     | 374    |
| [`crc32-v2`](https://crates.io/crates/crc32-v2) | 0.0.5   | 1 megabyte baseline    | 2,800,666 | 374    |

From the previous table, we see the following observations:

- For relatively short inputs (1 byte long), **[`crc32-v2`](https://crates.io/crates/crc32-v2)** is the fastest, taking only **2 ns/iter** with a throughput of **500 MB/s** to compute the crc32 value.
- For larger inputs like 1 kilobyte and beyond, **`crc32fast`** becomes highly efficient, achieving high throughput (up to 12,036 MB/s for 1 megabyte in the specialized variant).
- The performance of the **[`crc`](https://crates.io/crates/crc)** crate is significantly lower, especially for larger input sizes, with only 207 MB/s throughput.

In summary, `crc32-v2` is blazingly fast for short-input scenarios, while `crc32fast` dominates for larger data sizes.

## Memory Safety

**[`crc32-v2`](https://crates.io/crates/crc32-v2)** is memory safe.

Due to the use of SIMD intrinsics for the optimized implementations, this crate contains some amount of `unsafe` code.

In order to ensure memory safety, the relevant code has been fuzz tested using [afl.rs](https://github.com/rust-fuzz/afl.rs) with millions of iterations in both `debug` and `release` build settings. You can inspect the test setup in the `fuzz` sub-directory, which also has instructions on how to run the tests yourself.

On top of that, every commit is tested using an address sanitizer in CI to catch any out of bounds memory accesses.

Even though neither fuzzing nor sanitization has revealed any safety bugs yet, please don't hesitate to file an issue if you run into any crashes or other unexpected behaviour.

## Available feature flags

### `std` (default: enabled)

This library supports being built without the Rust `std` library, which is useful for low-level use-cases such as embedded where no operating system is available. To build the crate in a `no_std` context, disable the default `std` feature.

Note: Because runtime CPU feature detection requires OS support, the specialized SIMD implementations will be unavailable when the `std` feature is disabled.

### `nightly` (default: disabled)

This feature flag enables unstable features that are only available on the `nightly` channel. Keep in mind that when enabling this feature flag, you
might experience breaking changes when updating compiler versions.

Currently, enabling this feature flag will make the optimized `aarch64` implementation available.

## License

This project is licensed under either of

- Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or
  http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or
  http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this project by you, as defined in the Apache-2.0 license,
shall be dual licensed as above, without any additional terms or conditions.
