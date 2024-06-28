# NFT Marketplace - Testing

#### NFT Real Estate Marketplace testing with polkadot js as frontend

To interact with the NFT Real Estate Marketplace, all user must have already gone through the DID/KYC/KYB/AML (Kilt DID & Deloitte Verifiable Credentials) process (and be whitelisted) in order to be able to call the extrinsics of the marketplace.

**1.0 Listing Object**

The listObject function must be called by the Real Estate Developer in order to list an object on the marketplace.

The price must be selected as a parameter. This is the total price of the real estate object. Each NFT corresponds to 1% of this price. The property is divided into 100 NFTs.

The data is the metadata of the collection and the metadata of the individual NFT.

<figure><img src="https://lh7-us.googleusercontent.com/qWJ7cwSyMl-wqD-WXPsh-Rb2OhBdt_pcFDcL6XHfOK6q_Rcm4ADB-q3GLQDFq_Mv_5xarBUp8feSnIrfPlGwS_x7MQv3b7CNtgv9RYaWI6B44JMCcipTuq6TjEzGc0Web18mlYMQCCKYT2f2__kXrg" alt=""><figcaption></figcaption></figure>

\
**2.0 Listed Object**

Once the object is listed, the user has the option to upgrade the price of the object.

To do this, they must call the upgradeObject function.

The collectionId is the collection ID of the listed object.

The price is the updated price of the real estate object. All remaining listed NFT of the object will be updated accordingly.

<figure><img src="https://lh7-us.googleusercontent.com/35x4dZQnyUiz3-spg5c_QO1S7AB_peIppIAcyIUFf2PJzOFXbO_oVThoeF9ToiGmjYDkhhnRodfMt_ESPCpwEQCCOK8Hd-W6g8mcsw0xuHndggmcMuK4E1MyWUSyVmD0RcadFhB8xgVoPbKSotVKPQ" alt=""><figcaption></figcaption></figure>

\
**3.0 Buying process**

To buy NFTs, the real estate investor must call the function buyNft.

As parameters, the investor must specify the collectionID of the property and the number of NFTs he wishes to purchase.

<figure><img src="https://lh7-us.googleusercontent.com/dTP4b7Q8Vd6tZ_DSFpLm09twEUaifRFvAh7ssSPijdKC4SwSsadByluMtTMlidGYLCz9e9DdfBqLVks-SC5FeU66lmtZoHtYB-GY9UhAn_7fJsWq_QvHaehxmeEYM8Az-VjhhNukKKOmyaHcowt_rA" alt=""><figcaption></figcaption></figure>

\
**4.0 Object sold**

Once the property is sold, the SPV is created, the funds go to the real estate developer and the NFTs are sent to the investors/buyers.

The real estate investors can relist the NFTs on the marketplace by calling the listNft function.

The collectionID of the object and the itemID of the NFT that the investor wants to list are required as parameters for this function. The price is the price of the individual NFT.

<figure><img src="https://lh7-us.googleusercontent.com/-Mj_3iZELIBnviHwIZLNZrDugVMI7iBNLzl69OVaLlAuAXGj81uFWVkdbV3ONnAraEhLx_-E7-5Ph9OUtikSSy9ndq1JrfUvhxQGbuEfJe09K3MMNXoaIyKQudAPWVOL-9gnD6ST_RZb4uveFw3MBA" alt=""><figcaption></figcaption></figure>

\
After listing, the investor can upgrade the price by calling the upgradeListing function.

The collectionID of the object and the itemID of the NFT that the investor wants to upgrade are required as parameters for this function. The newPrice is the price of the individual NFT to which the investor wishes to upgrade.

<figure><img src="https://lh7-us.googleusercontent.com/pRzWJLHLe_IIAd--1aQJFbsYkbOgSvtKFMpGMUO9uHSm1RAGV8mktyiNPxQ_BpwgIHXtnobfY1yaJrTrCfhN-byrQqt21bx-Ou4N_3V7tMxhXt6sMl8MUII-81B-QyoseODHYELHmRYoWzCnKKShzA" alt=""><figcaption></figcaption></figure>

\
To remove the newly listed NFT from the marketplace, the investor can call the delistNft function.

The collectionID of the object and the itemID of the NFT that the investor wants to remove are required as parameters for this function.

<figure><img src="https://lh7-us.googleusercontent.com/om5aQV9ir9KdbSBdTBPbfnxlSCUUYEl-wAe4XaRVDfntE0BgHalDm1wvbPmH-gLi_T3RguaPr8dyYZG7opj9pcVZs2_E1uGGxm9F02HLl9rSb_X0nfgA6flNBg96wo92GPJLrnNCT8yCsYyGe9H8oQ" alt=""><figcaption></figcaption></figure>

\
**5.0 Buy a relisted NFT**

To buy a relisted NFT, the buyer must call the function buySingleNft.

The collectionID of the object and the itemID of the NFT that the buyer wishes to purchase are required as parameters for this function.

<figure><img src="https://lh7-us.googleusercontent.com/RpHSeuM9tgh4EEBecquZY9Dj9hzLYCoOXcPbOsHe5fSN_Ax5Uk3fDsQVOPxPFa4lMlcYV77bPP8C0GhU0lqUF_6zS98ktyWrcQZDXRBujqNXN8s4hzksegFBJdIzilzQK_1KaCQ-mb9ZB3Z5FUL6xw" alt=""><figcaption></figcaption></figure>
