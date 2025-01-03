# NFT Marketplace - Testing

NFT Real Estate Marketplace testing with polkadot js as frontend

To interact with the NFT Real Estate Marketplace, all user must have already gone through the DID/KYC/KYB/AML (Kilt DID & Deloitte Verifiable Credentials) process (and be whitelisted) in order to be able to call the extrinsics of the marketplace.

At the moment, an account must be added to the xcavate whitelist by calling the add\_to\_whitelist function in the xcavate\_whitelist pallet. This can only be done by the sudo account.

**1.0 Create new Region**

Before users can list objects on the marketplace, a region must be created with the sudo account.

This can be done by calling the createNewRegion function.

A region represents a country.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXfN9N25usprEuTCkAV7DKHY-j4x2dif0ooMh7_8svWYWRxgQjqbcth5qsLfXv4wTTNV-iZ5zksV9B4jZUKaqq5I8MYPVHqLUIMpntabJugMhQnAvj1O17JYc2zc34qgW3_kKzLybFxvR3-AlH8?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

**2.0 Create new Location**

Once a region has been created, a location must be created in this region with the sudo account.

To do this, the createNewLocation function must be called. The region is created as a parameter. The region is the region ID of the region. The location is the postcode of the location that should be created.

\
A location represents a city in the country.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXepxaDuaH9Pz6bYZw59UTgR_BYRgWfeNxqplT5AZD_NIH0omUSVYCoW_uzx9qbbkEAmFI9_9WoKtSUWB6L16kLTpTTT961ULUeAxIVp2I757qoW3Q5xDr52REalyuBMuqaDiMw7NhHU9km1dtxL001QxhA?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

**3.0 Listing Object**

The listObject function must be called by the real estate developer to list an object on the marketplace.

The region is the region ID of the country in which the property is located.

The location is the postcode of the city in which the property is located.

The tokenPrice must be selected as a parameter. This is the price of a single token.

The tokenAmount is the amount of token the property is divided in.

The data is the metadata of the nft that represents the property.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXctopF0J0MFFOQzMGcMbtI3I73J_SVToEa6uIQfVzBD_4INHuHWqbALSrCt3zNeAEh4s_l8qTqG5jwShdmydJYyRX5gPM4bgMjcCzsh84OzAJFGkX5up1QDQCP_h5U5DbI7qh1GFHYjm1gX13P0oTh4oYY?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

**4.0 Listed Object**

Once the object is listed, the user has the option to upgrade the price of the object.

To do this, they must call the upgradeObject function.

The listingId is the listing ID of the listed object.

The newPrice is the updated price of the real estate object.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXeg_8tWMoUE1GrhgGWm5zrnODVN2o27VybwXHUQHle20RHNrDmr3Z1k5JYc02Wm0nd5ioq3u3aqwQ5G4NG8JNyBW6Gnhjtd25b3xyY7KH5UcSLFXiHFrw0q7_imJAoVhpMBiZ_Y30xy-zjL5LP5rk3BPg?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

**5.0 Buying process**

To buy tokens the real estate investor must call the function buyToken.

As parameters, the investor must specify the listingId of the property and the number of tokens they wish to purchase.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXcNAe8uPmRxHVSmIeyzSGjKXEnPnwmSbsmogokC77M2CSbEarh8Rl5yukAmiPk-T1T4WDAIJytvV7OSDXeESB3hHNbHvfRAjR7_Lc6YDoSVZVpZVRiwwsi2aKXj8sibitjfiKkY4qVF8Tn-BW3GZ_LH3vs?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

**6.0 Lawyer process**

A lawyer must first be registered by the sudo account.

This is done by calling the registerLawyer function with the lawyer’s account ID as a parameter.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXc-Fj09Cx9WZgOVQDb57HJ1D4_wsm4zjlZb_Xim1qxIp2ApBVY6oRti_gBK3E_iWV-VoDJyjzR2EbXxLcg9FbMsPt3PLZ612wPK-M006Wb8C8BaW7MuR1CIvonvGYHrjyhaYdeOKo36eyNgmBh_Wlq0APVZ?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

Once all tokens of a property are sold, the lawyers can claim the property to manage the process.

Two lawyers are needed for a property transaction: one lawyer to represent the real estate investor’s side (SPV) and one lawyer to represent the real estate developer. This process requires two different lawyers, as it would be illegal for both sides to be represented by the same lawyer.

To take on the case, a lawyer must call the lawyerClaimProperty function.

The listingId is the ID of the listed object.

The legalSide is either RealEstateDeveloperSide or SpvSide.

The costs represent the lawyer’s fees, which cannot exceed 1% of the property price.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXdXw64Cf_D53e6DHu7Pw-y8DfT0UaMZHFC6SpJJiROrUX4XOs57vMCN0QOSzcuDAu4nwCF82Hk7wr928S9d1cpSIrMAMVFsbh9LoMLJedZDMpz7fJcF9YV1ZJz0mSlMvoknX_J5eLH06rV4EFYY5sVZMo65?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

Once the lawyer completes the legal work, he needs to call the lawyerConfirmDocuments function.

The listingId is the ID of the listed object.

Approve is a boolean value. A value of true means the documents are approved and the process can be finalised. A value of false means the process cannot be finalised, and the lawyer wishes to cancel the process.

Both lawyers need to confirm the process with true. Once this happens, the tokens will be distributed to the real estate investors. The fees will go to the treasury, and the lawyer's costs will be paid to the lawyer, including the 3% tax.

If one lawyer confirms with true and the other confirms with false, there will be a second round where the lawyers must confirm again. If both lawyers do not agree, the process will be rejected and not finalised.

This means that the NFTs and tokens will be burned. Additionally, the funds will be returned to the investors, including the 3% tax. Only the 1% fee will be paid to the lawyers to cover their costs, and the remaining fee will be sent to the treasury.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXc2mg523dhnhljVIO6uk2aBG7Gt6633nLtxQXs4wZoFfgiZcSQ8_A7lCXJ7RBDEQMxQsVxic0lzGKDJJ2DG_vIH8_HHiiTDoemszSNfUHWP_h8nLAQYmePC3K3_wKtq-gqE2bxXH0d9mgAzlAQXq_cpnQ89?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

**7.0 Object sold**

Once the SPV is created, the property is sold and the lawyers finalised the process, the funds go to the real estate developer and the tokens are sent to the investors/buyers.

The real estate investors can offer the tokens on the marketplace again by calling the relistToken function.

As parameters, the investor must specify the regionId, itemId of the nft of the property, price and quantity of tokens they wish to sell.

The property is represented by an nft and the nft is represented by the tokens, which are fractions of the nft. For this extrinsic, the seller needs the nft ID of the property.

The tokenPrice is the price of a single token.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXcJA5VXsgKXM-oUVLBRETaImFg9qnGxWTZgSHLnF7HHl5pJB6aQmTIY5KIpmN0KxKysPgzhbsQ5MF61hJpE_ez3Rom7w_b-0-AyrZwIlpop-Sf2mmmLLk0u240L56RYw6A4fal26gcZ_zrWHF-udiKJnf8?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

After listing, the investor can upgrade the price by calling the upgradeListing function.

The listingId of the listing is required as a parameter for this function. The newPrice is the price of a token.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXcPbgej4V4AWFhNVEFNXOpawlwgBUvfiIS5ASSdhUUGS9lDkZLJ9FFPUBNLTy9QhSvoQgIHJEAFvvQq_N9HVz0NGHwly61u4HqxmEWd8x7fMon0saFyKOG4216d6aiXFiBGTKN4oUdJMxWGlBL220hHo1Q?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

To remove the newly listed tokens from the marketplace, the investor can call the delistToken function.

The listsingId of the listing that the investor wishes to remove is required as a parameter for this function.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXe5LAV1x38caKGvzNIhItCv6cMAXsnvhAbI7v4bX3xdSVNUi5tN5kcqznfNjoGGbBi4fug0OBZW_N4KLv1bzVjyoZ0ZmqvhRlmQFXkf0RrmtwduMmyeioQilj_0j_weLUSzlk58UfB-PP9RqtcNjQ6proA?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

**8.0 Buy relisted tokens**

To buy newly listed tokens, the buyer must call the function buyRelistedToken.

\
The parameters required for this function are the listingId from the listing that the buyer wishes to purchase and the amount of token he wants to buy.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXftfZlWiZQnvwhJaO1QtiVz73a3RUyJ8yhAQwLcHOu2hrd_dH2cMJWCPNu-Dv5IvCHUSzHe3SbRRZKr-3GfqE88ouQoOqJxkv0Gy2E2-EQYJduLQxwY586c8j5zjbM5LYE7VNH5e074eucB24Ou05wYXN4?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

**8.0 Make Offer**

As an alternative a user can create an offer for a token listing by calling the makeOffer function.

\
As parameters the listing id of the token listing, the token price for a token and the amount of token the user wants to buy are required.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXdeUKXz2lxgGNcLJf4JZZ8FYzqO75DxudkY3EpPersgIADFjhhVNBYPCXQS3tktRxr3GCu-C8aD2A_vW2A7SFU9PV8eMlGdFZWf2wRcXvJrj5dDtFSfqYOuCk9SX55gd5JwzKGwB25sB_tjTu5g17whFRo?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

The seller of the token has now the option to either accept or reject the offer by calling the handleOffer function.

The listingId, the offerId and the enum Offer with the choices Accept or Reject are required as parameters.

\
In case the seller rejects the offer the funds that have been blocked from the user that made the offer will be released again.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXeNglqMDHM2YmG4McC1G5dhuTCOQpfKKDKqRBhN3efpnKqfVkCCd4Cjd_DmkInGk_YNJZpZGBFX6XY3gboALWzPFdchGfQMEjwU-pK2C8G73y3OW2zqFFK8lwunc1kvuXzu-phD08foljR9eaq_64g8IKM?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>

The user that made the offer can cancel the offer anytime as long as the seller did not handle the offer, by calling the cancelOffer function.

As parameters the listingId and the offerId are needed.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXdvq6lc6KfYi4IkT-CbXqom7BY61qmhrEmbXT3p6iKyAsD5b-H-MGGzGBHEgQhr52VgaQTAkOft8_GWsJPf7YlKfGRGy2qYEhpVFgAishZovuzpbLa7xuSVhuNHAWJ9vszvZNwz0Pk2edomK3aj2YmuHg?key=7f9wZvyo4duGyV394DhzbQ" alt=""><figcaption></figcaption></figure>
