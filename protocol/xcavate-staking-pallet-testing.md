# Xcavate Staking Pallet - Testing

**1.0 Stake**

To interact with the Xcavate Staking pallet, all user must have already gone through the DID/KYC/KYB/AML (Kilt DID & Deloitte Verifiable Credentials) process (and be whitelisted) in order to be able to call the extrinsics of the staking pallet.

\
To stake the token, the user must call the stake function. The amount of the token that the user wants to stake is required as a parameter.

<figure><img src="https://lh7-us.googleusercontent.com/D9drTdB8pFpr-MNC5qLNbD1Sa2N3vCxkXNYd4tSybw_9Huq3dDYeiITwfgzv5hwSeBedyytty3sgi4UuhG8_xh8FN_q4krwEOCftHom-Ssw6Q7cXK0SjwRgdeB--xyJ3V0_Z8MM4sGD9Yagvi5i3ZQ" alt=""><figcaption></figcaption></figure>

If enough tokens are available from the lending pool and the staking pool is not full, the user stakes their tokens and receives some rewards for their staked tokens. As the rewards are distributed in each block for testing purposes, a large number of tokens are required to ensure that some rewards are distributed. If the staking pallet is full, the staker must wait in the queue until a space becomes available.

To unlock their token, the user must call the unstake function. This requires that the token is not in the queue. The function requires the user's stakingIndex and the amount that the user wants to return as parameters.\\

<figure><img src="https://lh7-us.googleusercontent.com/0-ryoz_hoAbCKkPgtyVpbfxgC0f-JPm-JI6a_outPp8TNofdf0vyv05aX1Q-FJX1G7Gm0KXU0wcWCinXTXqgF2u4JkTACkvQusyW8HEf6ruUDEPoCgl-cQHITh2xlvZ57D4LbWYO16X3V8h8B6C8wg" alt=""><figcaption></figcaption></figure>

If the user has their token in the queue, they must call the withdrawFromQueue function to withdraw their token. The queue index and the number of tokens that the user wants to withdraw are required as parameters.

<figure><img src="https://lh7-us.googleusercontent.com/_0M_qWU2ZdfKSW_KIix2vWEgwacEjp8wtBgPy06OEyJWbWaHlg1SKIyoHlY9JlCtvc5wJ1OmzbtyyimFei0xLacgBefV4Rqey-_wgYpqNsHT_0QU2TUvKXksIHOhVAy_NgeHgOJeWglkMxwWTYmdWw" alt=""><figcaption></figcaption></figure>
