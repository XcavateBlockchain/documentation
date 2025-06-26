# Findings and Observations

#### **1. Access Control and Authorisation**

* Strong role-based authorization mechanisms in place.
* Proper use of Substrate’s built-in `ensure_signed`, `ensure_root`, and ownership checks.
* Multi-signature schemes recommended for high-value transactions.

#### **2. Cryptographic Integrity**

* Transactions and state changes are cryptographically verifiable.
* Metadata and pricing updates traceable through event logs.
* Protection against double-spending and token duplication confirmed.

#### **3. Common Vulnerabilities**

* **Reentrancy:** Mitigated by design – no nested calls or external contract dependencies in core functions.
* **Integer Overflows/Underflows:** Rust’s safe arithmetic and explicit checks confirmed.
* **DoS via High Gas:** Weight benchmarking validated reasonable limits under simulated loads.
* **Unauthorized Actions:** Tests confirm rejection of unauthorized marketplace actions.

#### **4. Resilience and Recovery**

* Clear rollback procedures and governance fallback mechanisms documented.
* System behavior under high load and edge conditions meets NIST availability requirements.
