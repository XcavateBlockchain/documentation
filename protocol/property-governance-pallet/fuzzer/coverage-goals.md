# Coverage Goals

Fuzzer should aim to cover:

* `submit_proposal()`
* `vote()`
* `finalize_proposal()`
* `reject_proposal()`
* All hooks: `on_initialize`, `on_finalize`
* Custom dispatchables (e.g., `delegate_vote`, `cancel_proposal`)
