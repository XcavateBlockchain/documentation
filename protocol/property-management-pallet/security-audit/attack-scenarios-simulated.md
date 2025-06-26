# Attack Scenarios Simulated

| Attack Vector                  | Simulated Outcome                                 | Mitigation Status |
| ------------------------------ | ------------------------------------------------- | ----------------- |
| Unauthorized property update   | Rejected due to owner mismatch                    | Resilient         |
| Flooding with registrations    | Bounded by runtime weights and storage size       | Resilient         |
| Bypass of `transfer_ownership` | Requires origin + current owner match             | Resilient         |
| Storage corruption via upgrade | `StorageVersion` and migration tests prevent this | Resilient         |
