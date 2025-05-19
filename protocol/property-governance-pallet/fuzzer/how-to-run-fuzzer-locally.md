# How to Run Fuzzer Locally

```
cargo test --package pallet-property-governance --features fuzzing
```

Or directly call a fuzz test:

```
cargo test test_fuzz_proposal_submission
```

You may also invoke from within the runtime test suite:

```
#[test]
fn test_fuzz_proposal_submission() {
    let result = fuzzer::generate_random_proposal();
    assert_ok!(PropertyGovernance::submit_proposal(origin, result));
}
```

