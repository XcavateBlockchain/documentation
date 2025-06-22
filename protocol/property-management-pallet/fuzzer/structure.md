# Structure

Located in:

```rust
#[cfg(test)]
mod fuzz_tests;
```

or optionally extracted into:

```rust
pub mod fuzzer;
```

#### Key Functions

| Function                          | Description                                                                         |
| --------------------------------- | ----------------------------------------------------------------------------------- |
| `generate_random_property()`      | Produces a randomized property struct with edge-case metadata and owners.           |
| `generate_random_transfer()`      | Simulates transfers to random recipients, including unauthorized or self-transfers. |
| `fuzz_property_lifecycle(n: u32)` | Executes n random property lifecycle operations (register, mutate, transfer).       |

