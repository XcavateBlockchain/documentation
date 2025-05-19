# How to Extend the Fuzzer

To add more test coverage:

1.  Add new generator functions in `fuzzer.rs`:

    ```
    pub fn generate_conflicting_proposal() -> Proposal {
        // Compose two mutually exclusive changes
    }
    ```
2.  Add new fuzz test case:

    ```
    #[test]
    fn test_fuzz_conflict_detection() {
        let proposal = generate_conflicting_proposal();
        assert_err!(
            PropertyGovernance::submit_proposal(origin, proposal),
            Error::<Test>::ConflictingProposal
        );
    }
    ```
3.  Automate with looped inputs:

    ```
    for _ in 0..100 {
        let proposal = generate_random_proposal();
        let _ = PropertyGovernance::submit_proposal(origin.clone(), proposal);
    }
    ```

