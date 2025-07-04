# How to Run the Fuzzer

Run default fuzz tests using:

```bash
cargo test --package pallet-property-management --features fuzzing
```

Targeted fuzz execution:

```bash
cargo test test_fuzz_unauthorized_property_transfer
```

You can loop multiple randomized cases:

```rust
#[test]
fn fuzz_bulk_property_transfer() {
    for _ in 0..50 {
        let action = fuzzer::generate_random_transfer();
        let _ = apply_property_transfer(origin.clone(), action);
    }
}
```

