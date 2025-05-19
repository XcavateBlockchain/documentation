# How to Run Fuzzer Locally

```
cargo test --package pallet-nft-marketplace --features fuzzing
```

Or directly call a fuzz test:

```
cargo test test_fuzz_listing_underflow
```

You can run multiple iterations via:

```
#[test]
fn test_fuzz_multiple_nfts() {
    for _ in 0..50 {
        let nft = fuzzer::generate_random_nft();
        assert_ok!(NFTMarketplace::mint(origin.clone(), nft));
    }
}
```

