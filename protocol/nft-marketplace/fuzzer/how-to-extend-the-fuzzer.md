# How to Extend the Fuzzer

To add additional coverage:

1.  Add test case generators:

    ```
    pub fn generate_high_value_listing() -> Listing {
        // Max value, long duration, uncommon currency
    }
    ```
2.  Add targeted fuzz test:

    ```
    #[test]
    fn test_fuzz_high_value_listing() {
        let listing = generate_high_value_listing();
        assert_ok!(NFTMarketplace::list(origin, listing));
    }
    ```
3.  Loop over randomized sequences:

    ```
    for _ in 0..100 {
        let action = generate_random_market_action();
        let _ = apply_random_market_action(origin.clone(), action);
    }
    ```

