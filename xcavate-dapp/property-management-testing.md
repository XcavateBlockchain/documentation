# Property Management - Testing

Prerequisites:

* Users must complete DID/KYC/KYB/AML (Kilt DID & Deloitte Verifiable Credentials) process and be whitelisted.
* Accounts must be whitelisted by the sudo account using the add\_to\_whitelist function in the xcavate\_whitelist pallet.

Setup:\
Follow the steps in the NFT Marketplace documentation up to point 5 ("Buying Process"). A property must be listed and sold.

#### 1. Add Letting Agents

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

#### 2. Letting Agent Deposit

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

\


