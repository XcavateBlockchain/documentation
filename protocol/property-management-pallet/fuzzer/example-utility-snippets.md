# Example Utility Snippets

### Example Utility Snippets

```rust
pub fn generate_oversized_metadata_property() -> Property {
    Property {
        id: random_hash(),
        owner: AccountId::random(),
        metadata: vec![b'A'; 1024], // Exceeds max metadata length
        valuation: u64::MAX,
    }
}
```

```rust
[test]
fn test_fuzz_invalid_transfer() {
    let property = generate_random_property();
    let unauthorized = AccountId::random();
    assert_err!(
        PropertyManagement::transfer_property(Origin::signed(unauthorized), property.id, property.owner),
        Error::<Test>::NotPropertyOwner
    );
}
```

