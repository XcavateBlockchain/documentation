# Xcavate Community Loan Pallet - Testing

**1.0 Add Committee Members**

To add members to the voting committee, the sudo account should call the addCommitteeMember function. The accountId of the new member is required as a parameter.

<figure><img src="https://lh7-us.googleusercontent.com/ksH-17b8ybwFWUaGRdVtTmDMDDqlEBIbXykr9YZNOxWPr33cq0PU7JCCohsbuyGbKh9ssEWrAuGJAdelp35a0eUmvo0_47j8Pui1JSFkoEKLXdahaJh7-TP7vddFOSS4itYYxvecLlB8th991jCMPA" alt=""><figcaption></figcaption></figure>

**2.0 Loan application process**

To interact with the Loan Pool pallet, all user must have already gone through the DID/KYC/KYB/AML (Kilt DID & Deloitte Verifiable Credentials) process (and be whitelisted) in order to be able to call the extrinsics of the pallet.

\
The voting time is currently only a few blocks for test purposes.

To apply for a loan, the developer must call up the propose function.

The loan amount, the accountId of the loan recipient (beneficiary), the developerExperience of the developer in years and the term of the loan in months (loanTerm) are required as parameters.

<figure><img src="https://lh7-us.googleusercontent.com/FZcpjIj_KD4qTXCJ-9812-eLt1_Vx8UC1ytITjl47ugbTK9Lxc4cBtmRa2o1c0C9r05bfO0_zcyvgfK9qdf0U1jMRmZNMoENImXSujh3iJXHwoFgb_8GXoOuEnhPNiOmbpRa0Kat272xXJ0LYEm37Q" alt=""><figcaption></figcaption></figure>

Once an application has been submitted, the voting committee can vote for or against the loan. First, a committee member must set the milestones in which the loan is to be distributed. This can be done by calling the setMilestones function.

\
The index of the proposal (proposalIndex) and a vector with the various milestones are required as parameters. The various milestones are represented by the percentage that is to be paid out for each milestone.

<figure><img src="https://lh7-us.googleusercontent.com/_Bce9CNTlniFmcrYP5hCxS7t1Fw71ILQuUYNZoz7r265DhMrNPiuX-lexZUAADrhIUMNY_XsAIZQvZyWH2yXIL6ELo25Z0ItfLPXG-_NbyIT0LmicE4aagxgUC6qVp85GXX_TJgxj980w4Mlfr0DAg" alt=""><figcaption></figcaption></figure>

Once the milestones are set, other committee members can vote for or against the proposal by calling the voteOnPropsal function.

\
As parameters, the function requires the proposalIndex and the vote, which is an enum with Yes or No as the value.

<figure><img src="https://lh7-us.googleusercontent.com/nF48iIrW7rrc2G-FcRQI5qR9iRHp_lLMzv86f5NY-9HN7nlFpeLhxWpCPtxcLjI7dGxMEzNpZ8jkxql5fqiGKDroyIolq75Iewzyrd-xzqX2sAc2RCQB-p0AAXCOQyfVVg154Lg5oCzofrasocuodA" alt=""><figcaption></figcaption></figure>

**3.0 Loan approved**

If the loan is approved, the real estate developer can withdraw funds from the loan pool. Only the beneficiary wallet from the loan application can withdraw funds.

\
The function requires the parameter loanId as the index of the loan and the amount that the developer wants to withdraw from the loan. Only the amount that was activated in the first milestone can be withdrawn.

<figure><img src="https://lh7-us.googleusercontent.com/whq-7ZgBPLdmj6dTP2JDsPu-35iISihQ5cMZcRHmb4-qn-2dHTVEyicJbjLe8EHFwJ-jH8nG00HuVtRhp4abgsb6Uiy878QDuMy7amQVWYFmfn87RwUAruC7v80N6GOW0pkSS4ZamE7aqEX9JLlQkw" alt=""><figcaption></figcaption></figure>

The developer can make a proposal for the next milestone by calling the proposeMilestone function. This function can be called up by any user who is on the whitelist.

\
Only the loanId is required as a parameter.

<figure><img src="https://lh7-us.googleusercontent.com/ij8dAF0Ps3Kl6cEsWEllb3gHBQa2xwRiTuwrFLoIGB0J4ac_n1yn9yCshCGYBa0HxGUfEjilPEK3r-8TegIQcc7-7bMtGuuv9IMZ4du5Cuywm3U0G44Q6NzTMe9dHGwpEBmF7MVOsO9F32FSCayLnQ" alt=""><figcaption></figcaption></figure>

The committee can vote on the milestone proposal by calling voteOnMilestoneProposal.

\
The function takes the parameter proposalIndex which is the proposal index of the milestone proposal, and it takes the vote enum which can be yes or no.

<figure><img src="https://lh7-us.googleusercontent.com/QfqSo48egyCN6SAawcY6TQUykVMXTGO-3cYWQTsC2HieSm73y5vdORR2GmjYala9mwVtdyFbPZ_kDgPz5oCvpvUVQKhFVN39B4Tzqb2kSHufQUX1dJTQNTjVGUGgGHeGpkWe_uyA-p-b_UqzAbNR4A" alt=""><figcaption></figcaption></figure>

Once the milestone vote passed the next charge of the next milestone will be available for the real estate developer.

To repay the loan, the developer can simply call the repay function.

\
This function requires the loanId and the amount that the developer wants to repay as parameters. All users who are on the whitelist can call the function and repay a loan for someone else.

<figure><img src="https://lh7-us.googleusercontent.com/gmgusmBKv6TnjH6ThcY_IbJYLbML9bIJixyx9Uh_ShdTU4yY_9qdp9NWcbHFp4mDClULj7mE5-opf-fyQt2kg8p1OTx7_kgsMC1O6YuI2p0LFJJ7-PkFxzLvQPgAu9zNWkoxzX3sbZnWZYW6mWguyA" alt=""><figcaption></figcaption></figure>

Once the loan has been repaid in full, the developer can call the proposeDeletion function to request the deletion of the loan.

\
This function only requires the loanId as a parameter.

<figure><img src="https://lh7-us.googleusercontent.com/61n5Kk0BgDMrOdczPJ8Wjwmk44K5j7x5urd8OZeW5bokkhchiYy6EGNPc2F0e_zg0uIHnDr2sN7zABeEKWtJKWSkA519clD7gzFUAAxFXGzAPYLwwCiM8TCeKf0THT46bSr3_rxUW31sa47Fcb0NCA" alt=""><figcaption></figcaption></figure>

Here too, the voting committee must vote on the deletion proposal by calling the voteOnDeletionProposal function.

\
The function receives the proposalIndex parameter, which specifies the proposal index of the deletion proposal, and the enum vote, which can be yes or no.

<figure><img src="https://lh7-us.googleusercontent.com/nHcslH6Az3GK2GbmoXYodDVEmm_xNNjmKTthuoqN2oFbm72j8CA16UZ4XHQKr3vSklMKlAQLnv7ZoKowtIpqWnpYABz69ao3Q1dXfRSsj5i135_H3W5JHl-QuPfHtQxCL0DdphbfhNoyjXQCYZMJFw" alt=""><figcaption></figcaption></figure>

If the vote was successful, the loan is deleted.
