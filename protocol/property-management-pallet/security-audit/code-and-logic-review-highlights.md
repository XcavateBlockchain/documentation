# Code and Logic Review Highlights

**Dispatchable Calls (Core Audit Focus)**

| Call                   | Security Risk                        | Mitigation                              |
| ---------------------- | ------------------------------------ | --------------------------------------- |
| `register_property()`  | Malformed input / unauthorized entry | Bounded input, requires `Signed` origin |
| `update_property()`    | Tampering by non-owner               | Uses `ensure_owner()` check             |
| `transfer_ownership()` | Reassignment by spoofing             | Origin match + storage verification     |
| `deprecate_property()` | Unauthorised deletion or misuse      | `ensure_root()` required                |



#### Error Handling and Hardening

* All extrinsics return typed `Result`
* Invalid state transitions explicitly mapped to `Error::<T>::*`
* `panic!()` is avoided in favour of graceful error returns



#### Event Logging

Every extrinsic action emits an event. Events are:

* Typed and namespaced: `Event::PropertyRegistered`, etc.
* Indexed for external systems to query
* Compliant with audit logging under FCA SYSC guidelines
