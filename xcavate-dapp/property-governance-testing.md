# Property Governance - Testing

Prerequisites:

* Users must complete DID/KYC/KYB/AML (Kilt DID & Deloitte Verifiable Credentials) process and be whitelisted.
* Accounts must be whitelisted by the sudo account using the add\_to\_whitelist function in the xcavate\_whitelist pallet.

Setup:\
Follow the steps in the NFT Marketplace documentation up to point 5 ("Buying Process"). Also, follow the steps in the Property Management documentation up to point 4 ("Distribute Income"). A property must be listed, sold, and assigned a letting agent.

### 1. Propose

**Purpose:**

Letting agents propose funding for property-related expenses.

**Function: propose**

* Called by: Letting agent
* Parameters:
  * assetId: ID of the property
  * amount: Amount of funds requested
  * data: Details of the proposal

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXe8Xr2wHlH0xABYDhKcql17SagHdPHP-keGadUMHuxxH2oNt1HKvLruIl14rgzSDJN3uRs9RstM-FgG4IdS8Zfng8N6VdTL_jQIWBY4tdPx7K92J2JEogbjWHTi_PaZtdLyyR3IzVsnCqhY0YqwjFw7Nqo?key=2-QFkm8ErZXNIzbRKvUpWw" alt=""><figcaption></figcaption></figure>

Process:

1. The letting agent initiates a proposal for necessary expenses (e.g., repairs).
2. Proposals need at least 50% approval to pass.

* Proposals below 500 tokens (per runtime config) are executed directly.

Note: Proposals allow for transparent and democratic decision-making regarding property expenses.

### 2. Voting on Proposal

**Purpose:**

Token holders vote on proposals based on their token holdings.

**Function: voteOnProposal**

* Called by: Token holder/property owner
* Parameters:
  * proposalId: ID of the proposal
  * vote: Enum (yes or no)

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXdJnqzxe300bXxRkJtIiEmeOOrNV3QHxFqcxRNZvGA1Scq_LfuOhfIFU33tHUJWMwDMySFeol3tFINh1-FDy4P62F6aOl7qRUDps05ioIEBhGkHE7NQlb3DbZ-IUO9Tcdnd_cwEkNMdGXgWuI9qSdm0sApv?key=2-QFkm8ErZXNIzbRKvUpWw" alt=""><figcaption></figcaption></figure>

Note: View proposal status: Query proposals in chain state. ProposalId starts at 1.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXcyeHjrtwuO92Z7ScbPy_qTeGukvh_XpAVEkdk3LQJ3lY9jW6ZGWDQxrzoCCCBbAszyyTIviVGoyu-2rMVGomYvmgwGtXVRbW6LyIcuXWKPEnyYq03tb8mMI622RiNtOxsd7wkW4_6w7xW10R6zs1dbtwn5?key=2-QFkm8ErZXNIzbRKvUpWw" alt=""><figcaption></figcaption></figure>

Note: View votes: Query ongoingVotes in chain state.\


<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXfrtEIwRtOS0UEilKdLBW_Ti8ARzqIQ3D9d7M1xcv2UMOzBT5eY-96IPXM9Q15HrxsF_RCiK3Rjk4hxVu04ckuz8lWKcYux_H9eCW854LaoqenIh9-Rvz1ssFfouCT5qSYyz90aFU_VifPIxM1BeKUBpQzS?key=2-QFkm8ErZXNIzbRKvUpWw" alt=""><figcaption></figcaption></figure>

Outcome:

* Accepted proposals transfer the amount to the letting agent, deducted from real estate reserves.
* If reserves are insufficient, the deficit is added to real estate debt, to be paid from future income.

Note: Voting ensures that token holders have a say in significant property-related decisions.

### 3. Challenge Letting Agent

**Purpose:**

Property owners express distrust and potentially replace a letting agent.

**Function: challengeAgainstLettingAgent**

* Called by: Property owner
* Parameters:
  * assetId: ID of the property

Note: Challenge process has four stages:

* Stage 1: Vote to express mistrust.
* Stage 2: Letting agent defends (no voting).
* Stage 3: Vote to slash the letting agent.
* Stage 4: Vote to change the letting agent.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXeqDfQWzXgpNpUzDlaNDOoGrp9yjrTyw_bS5pSrWoXfOinD1ZPU9pF5Izy3IZO_nKETG6OlrHEgL7XhbpJcY2M-ajamCDOfBBfFfMTSZyuT0slKzpKLXCpBnWgMmd1oXyUdILjZ_ixAVJhAdfsf1a8N8ZlW?key=2-QFkm8ErZXNIzbRKvUpWw" alt=""><figcaption></figcaption></figure>

Note: The challenge process ensures accountability and provides a mechanism for addressing grievances against letting agents.

### 4. Vote on Challenge

**Purpose:**

Property owners vote on each stage of a challenge against a letting agent.

**Function: voteOnLettingAgentChallenge**

* Called by: Property owner
* Parameters:
  * challengeId: ID of the challenge
  * vote: Enum (yes or no)

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXdgsTmKZf-2s60acDYBfWVMCKD3g_JwDtde40Lamzgc1clDi_U_20MG7qnTr3g_kxL9n71RIHNYBtWdLoaqI1xUQr8PS9yWVh3n0gtqDH6Tf3ZSeJfqFkRipkwKj7pn53Zp-PAPBJgyX0-lKVtcUosjNM0?key=2-QFkm8ErZXNIzbRKvUpWw" alt=""><figcaption></figcaption></figure>

Note: View challenge status: Query challenges in chain state.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXd8V1yVmvasYXfRTxU-DKKF2MUDYgIqZ-2hWFlErq87Jwz_99AmkTG4qNMl8ug-oDcwQPFF4I4R95w7Db2oTPoFnKEMPQDWlgPJVVFO9-fJisjERyT4-3kG6h9OMEehXZtrEOrzeUGlIzwXu4eWL3y9XJjq?key=2-QFkm8ErZXNIzbRKvUpWw" alt=""><figcaption></figcaption></figure>

Note: View current voting: Query ongoingChallengeVotes in chain state, specifying challengeId and current stage.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXcgz-zuYeyi5T0u4aMHmumpioowHM8liq3H4U7vmsnNU9lkxErRDBl7L81z3SO1Xee44oH_ixraKJQ-Vbnxsvvmcy1RlshfUG3et91HOPAew2YVnPBzP-zksAqMmCV6UP6fMonPGvhj77cLtj8LbuQLzVE?key=2-QFkm8ErZXNIzbRKvUpWw" alt=""><figcaption></figcaption></figure>

Note: Voting on challenges allows property owners to take collective action against underperforming or unethical letting agents.\
