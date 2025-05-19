# File Layout Suggestion

```
pallet-nft-marketplace/
├── src/
│   ├── lib.rs
│   ├── fuzzer.rs       # ← Fuzzing logic lives here
│   └── mock.rs
└── tests/
    └── fuzz_tests.rs   # Optional: integration fuzz tests
```

