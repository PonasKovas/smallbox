# `smallbox`

[![Build Status](https://travis-ci.org/andylokandy/smallbox.svg?branch=master)](https://travis-ci.org/andylokandy/smallbox)
[![crates.io](https://img.shields.io/crates/v/smallbox.svg)](https://crates.io/crates/smallbox)


`Small Box` optimization: store small item on the stack or fallback to heap for large item.

## [**Documentation**](https://andylokandy.github.io/smallbox)

# Usage

First, add the following to your `Cargo.toml`:

```toml
[dependencies]
smallbox = "0.6"
```

Next, add this to your crate root:

```rust
extern crate smallbox;
```

If you want this crate to work with dynamic-sized type, you can request it via:

```toml
[dependencies]
smallbox = { version = "0.6", features = ["coerce"] }
```

Currently `smallbox` by default links to the standard library, but if you would
instead like to use this crate in a `#![no_std]` situation or crate, you can request this via:

```toml
[dependencies.smallbox]
version = "0.6"
features = ["coerce"]
default-features = false
```


# Feature Flags

This crate has the following cargo feature flags:

- `std`
  - Optional, enabled by default
  - Use libstd
  - If `std` feature flag is opted out, `alloc` crate
    will be linked, which requires nightly rust.

- `coerce`
  - Optional
  - Require nightly rust
  - Allow automatic coersion from sized `SmallBox` to unsized `SmallBox`.


# Unsized Type

There are two ways to have an unsized `SmallBox`: Using `smallbox!()` macro or coercing from a sized `SmallBox` instance.

Using the `smallbox!()` macro is the only option on stable rust. This macro will check the types of the expression and
the expected type `T`. For any invalid type coersions, this macro invokes a compiler error.

Once the feature `coerce` is enabled, sized `SmallBox<T>` can be coerced into `SmallBox<T: ?Sized>` if necessary.

# Example

Eliminate heap alloction for small items by `SmallBox`:

```rust
use smallbox::SmallBox;
use smallbox::space::S4;

let small: SmallBox<_, S4> = SmallBox::new([0; 2]);
let large: SmallBox<_, S4> = SmallBox::new([0; 32]);

assert_eq!(small.len(), 2);
assert_eq!(large.len(), 32);

assert_eq!(*small, [0; 2]);
assert_eq!(*large, [0; 32]);

assert!(small.heaped() == false);
assert!(large.heaped() == true);
```

## Unsized type

```rust
#[macro_use]
extern crate smallbox;

use smallbox::SmallBox;
use smallbox::space::*;

let array: SmallBox<[usize], S2> = smallbox!([0usize, 1]);

assert_eq!(array.len(), 2);
assert_eq!(*array, [0, 1]);
```

With `coerce` feature:

```rust
use smallbox::SmallBox;
use smallbox::space::*;
 
let array: SmallBox<[usize], S2> = SmallBox::new([0usize, 1]);

assert_eq!(array.len(), 2);
assert_eq!(*array, [0, 1]);
```

`Any` downcasting:

```rust
#[macro_use]
extern crate smallbox;

use std::any::Any;
use smallbox::SmallBox;
use smallbox::space::S2;

let num: SmallBox<Any, S2> = smallbox!(1234u32);

if let Some(num) = num.downcast_ref::<u32>() {
    assert_eq!(*num, 1234);
} else {
    unreachable!();
}
```


# Capacity

The capacity is expressed by the size of type parameter `Space`,
regardless of what actually the `Space` is.

The crate provides some spaces in module `smallbox::space`,
from `S1`, `S2`, `S4` to `S64`, representing `"n * usize"` spaces.
Anyway, you can defind your own space type
such as a byte array `[u8; 64]`.

# Benchmark

The test platform is Windows 10 on Intel E3 v1230 v3.

```
running 6 tests
test box_large_item                  ... bench:         104 ns/iter (+/- 14)
test box_small_item                  ... bench:          49 ns/iter (+/- 5)
test smallbox_large_item_large_space ... bench:          52 ns/iter (+/- 6)
test smallbox_large_item_small_space ... bench:         106 ns/iter (+/- 25)
test smallbox_small_item_large_space ... bench:          18 ns/iter (+/- 1)
test smallbox_small_item_small_space ... bench:           2 ns/iter (+/- 0)

test result: ok. 0 passed; 0 failed; 0 ignored; 6 measured; 0 filtered out
```


# Contribution

All kinds of contribution are welcome.

- **Issue** Feel free to open an issue when you find typos, bugs, or have any question.
- **Pull requests**. Better implementation, more tests, more documents and typo fixes are all welcome.


# License

Licensed under either of

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.