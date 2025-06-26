# Code and Logic Review Highlights

Dispatchable Calls (Core Audit Focus)

| Call                   | Security Risk                        | Mitigation                              |
| ---------------------- | ------------------------------------ | --------------------------------------- |
| `register_property()`  | Malformed input / unauthorized entry | Bounded input, requires `Signed` origin |
| `update_property()`    | Tampering by non-owner               | Uses `ensure_owner()` check             |
| `transfer_ownership()` | Reassignment by spoofing             | Origin match + storage verification     |
| `deprecate_property()` | Unauthorized deletion or misuse      | `ensure_root()` required                |
