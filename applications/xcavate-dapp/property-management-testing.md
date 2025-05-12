# Property Management - Testing

Prerequisites:

* Users must complete DID/KYC/KYB/AML (Kilt DID & Deloitte Verifiable Credentials) process and be whitelisted.
* Accounts must be whitelisted by the sudo account using the add\_to\_whitelist function in the xcavate\_whitelist pallet.

Setup:\
Follow the steps in the NFT Marketplace documentation up to point 5 ("Buying Process"). A property must be listed and sold.

### 1. Add Letting Agents

**Purpose:**

Add letting agents to a location (city) to manage properties.

**Function: addLettingAgent**

* Called by: Sudo account
* Parameters:
  * region: ID of the country
  * location: Postcode of the city
  * lettingAgent: Account ID of the letting agent

Note: Adding letting agents ensures that there are authorized individuals available to manage the properties in different locations.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXegtvWLS_-sproaeVGQ_8juagjZfp5LKApuRHPgA4bacBcvcE7PGS9sCrMVuxDuhAuyHXszXmiL_WYuVvRXgUn51vogE-knZAUiuf_ZUtCT650MakEZKhkHrwKCyi3Vhk_bxdtv2IHhhUZmHTL3yZDzh6k?key=6lwevPiqcH3wgshT580shA" alt=""><figcaption></figcaption></figure>

### 2. Letting Agent Deposit

**Purpose:**

Letting agents deposit collateral to become active.

**Function: lettingAgentDeposit**

* Called by: Letting agent
* Parameters: None

Note: This deposit acts as a form of security and ensures the letting agent's commitment to their responsibilities.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXd_t1dbHMZaOjdkhj1LPxiIo4rCvvKk8PebitSe6H40boAWbHQancgQfDe6RgpQjb9zwGFW7RoLzVr6a1sUJkkjVfUdg-poLt6BOL1lml_4v2de_CR-bzTkLArBIXXMXhE2HyBiB1k_Y1LZo3RjthzCB78?key=6lwevPiqcH3wgshT580shA" alt=""><figcaption></figcaption></figure>

**Additional:**

Letting agents can be added to more locations by the sudo account using the addLettingAgentToLocation function.

* Parameters:
  * location: Postcode of the city
  * lettingAgent: Account ID of the letting agent

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXd7OPURbcoLQzu_2aEEskveKuXp5p0DteUgMT4yEprjzLFh97-pj1uyQjlBWLlV6DI2VrrEzc51F2U7xyP0NhpT4Duv68Jkwgnd2Soz7pQ8WFtVAgq0VQxnuuPOaQhxL_tbFd6Ur_OgroC50an0LlrcCtiD?key=6lwevPiqcH3wgshT580shA" alt=""><figcaption></figcaption></figure>

### 3. Set Letting Agent for Property

**Purpose:**

Assign a letting agent to a property.

**Function: setLettingAgent**

* Called by: Any token holder of the property
* Parameters:
  * assetId: ID of the property

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXcgF2tCX3tPJdOL044oFx-8MVrqBUzafCGD_L_64YG0-hagVH08PK9XPRNhEXFo6eYFqL475nBWyo8_RYAEOI8P8jkJSOqwD646qRImETGBEqMnzH_ioN9UX3CfUH2v3Q6LoiOUq_8BxRHMaU8AnmwH0dE?key=6lwevPiqcH3wgshT580shA" alt=""><figcaption></figcaption></figure>

### 4. Distribute Income

**Purpose:**

Distribute rental income to token holders after reserving a certain amount for property reserves.

**Function: distributeIncome**

* Called by: Assigned letting agent
* Parameters:
  * assetId: ID of the property
  * amount: Amount of funds to distribute

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXeLVESVYJNofJisxhnDa6NvQotIAvvDlhDvEq1fdPj0i-mb7jR5Hg_1ynArvXnT2i6f8BHW4rxA8QMa_9h8NIp1yQeZLdxkk9yqvL_F5nsd1uSwONh6HSNhXqucsc1t4XudkE4c4nwYqIWHkCiyy3UKGMt9?key=6lwevPiqcH3wgshT580shA" alt=""><figcaption></figcaption></figure>

Process:

1. The letting agent initiates the distribution of rental income.
2. A certain amount is first allocated to property reserves.

* Check property reserves using the propertyReserve chain state query.

3. Once property reserves are filled, the remaining income is distributed to token holders in proportion to their token holdings.\


<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXcJF5WcLLws2OoxO31d_jx1gKkS4xe4w_uu8exO1lxRT3wN_DhkvccQY_yjSRcYRQu7A_WCLWQ4-pgZjCuoAkguWqjnldDstgLhS0Q5-vaE3HIF7rMWatyW95DH6JWbts-LLbivBxrfpS-VwkmRrsHPGRSP?key=6lwevPiqcH3wgshT580shA" alt=""><figcaption></figcaption></figure>

Handling Debts:

* If a property has debts, these must be covered before any income is distributed to token holders.
* Debts can arise from:
  * Governance proposals: When a proposal's requested amount exceeds the available reserves, creating a debt.
* Debt Repayment:
  * Any new income will first go towards repaying the outstanding debt.
  * After debts are fully repaid, any remaining income is allocated to property reserves and then distributed to token holders.

Note: Proper management of reserves and debts ensures financial stability for the property and fair distribution of income to token holders.

### 5. Withdraw Funds

**Purpose:**

Token holders withdraw their share of funds.

**Function: withdrawFunds**

* Called by: Token holder (for themselves)
* Parameters: None

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXdABLCnhDU_RajHylmwDUlk6X8LGJh5X40wr-eCDPY5eNc9opmaeCAa7evC3nuD0lG2ex0Rso7pJQJm0yDuladGnESqvcU-muCCCjw_xe9qgjcNSSoN1cuX8C235k_xdYYnmjzh2BDH4RLBryEysI60flha?key=6lwevPiqcH3wgshT580shA" alt=""><figcaption></figcaption></figure>

Note: View stored funds using the storedFunds chain state query for the account ID of the token holder.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXe_g3kz0C0lLwfkZo-51MsiNPmDkXX-cYLY7mYcqjFoB8wKk3kp8iGBlhDWIK0vnsyMVfYPu9y9eJD66g1T7Xzy2zBMyVKW_XNYoOkf4_j1ELbv3xhBZ1wE1f1Jma8jve3or55es6xIjI77z_iDWeh3SJ1b?key=6lwevPiqcH3wgshT580shA" alt=""><figcaption></figcaption></figure>
