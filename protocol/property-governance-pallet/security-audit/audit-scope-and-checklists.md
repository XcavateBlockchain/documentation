# Audit Scope and Checklists

#### **1. Identification & Risk Analysis (NIST: Identify)**

* [ ] Catalog assets: property records, ownership keys, governance actions.
* [ ] Assess potential attack vectors (e.g., unauthorized transfers, reentrancy).
* [ ] Map data flows and dependencies within the pallet.
* [ ] Evaluate cross-module interactions and runtime upgrade risks.

#### **2. Access Controls & Authorization (NIST: Protect)**

* [ ] Validate use of `ensure_signed`, `ensure_root`, and `Origin` checks for privileged actions.
* [ ] Prevent unauthorized ownership transfers, dispute submissions, or governance actions.
* [ ] Implement rate limiting and throttling for high-volume operations.
* [ ] Ensure least-privilege principles in all access control flows.
* [ ] Multi-factor checks for sensitive actions (if integrated with external identities).

#### **3. Data Integrity & Confidentiality (NIST: Protect)**

* [ ] Confirm cryptographic integrity of storage keys and runtime hashes.
* [ ] Prevent tampering with property metadata, ownership states, and governance proposals.
* [ ] Validate correct state transitions under concurrency or stress tests.
* [ ] Encrypt sensitive off-chain data references (e.g., property documents stored externally).

#### **4. Anomaly Detection & Logging (NIST: Detect)**

* [ ] Integrate logging hooks for critical actions (mint, transfer, dispute).
* [ ] Detect unauthorized or replayed transactions using nonce or signature checks.
* [ ] Monitor runtime events for abnormal frequency or outliers.
* [ ] Alert and notify governance leads upon detection of critical anomalies.

#### **5. Incident Response & Recovery (NIST: Respond/Recover)**

* [ ] Define response workflows for detected breaches or governance disputes.
* [ ] Implement rollback or slashing mechanisms for fraudulent governance actions.
* [ ] Ensure backup and restoration procedures for runtime state (where applicable).
* [ ] Document and review post-incident lessons learned.
