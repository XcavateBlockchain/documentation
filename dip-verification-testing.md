---
description: Explanation and Configuration on Substrate side
---

# DIP verification testing

The Decentralized Identity Provider (DIP) pallet enables a Substrate-based chain provider (i.e. KILT Peregrine) to bridge the identities of its users to other connected chain consumers (i.e. Xcavate chain) trustlessly. A consumer chain can connect to a provider chain if there is a way for the consumer chain to verify proofs about parts of the state of the provider chain.

DIP verification is achieved through a Substrate pallet structure, which has been added to the runtime of the consumer chain. This pallet code then performs an origin check in the pallet’s config, enabling access to the DID identifier of the submitter, so its origin can be accessed from the extrinsic (example below).

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXdGF46pvAlI7hy-88I8Iev3Ad1qNhaIWIsJ4MqerNPmvX7jaiIThlrvGD4lqiciK3iMtKJl2N5H9we5qGdzTmtaE0mwyGrNabh8sVH7tBOLiwBGojmccwsD-2E8Aeg3ZXsHBr3tMUDdr_dosrcjbnsQdtVR?key=HzgHVXYUZN16cG22fhmMVg" alt=""><figcaption></figcaption></figure>

The origin is created after the identity proof has been successfully verified by the proof verifier.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXceD82uJqRRsgwd3J0iiA8EHa2nk45hLV60uHLv622pKUMSkwk2JfUyq7qdqm9CZeft4orku2yYTvZ_E3EqtRgeQ6DvEBjoaB61hHfAsnXQ_JFfdwg-_hlKQXge3d6AjwXMrz0sxdYcQSTriSpxyIcDaCg?key=HzgHVXYUZN16cG22fhmMVg" alt=""><figcaption></figcaption></figure>

### Testing

Follow the below steps in order to call the list\_object function with a verified DID. This function can only be called if the origin has a verified DID.

1\. Clone DIP SDK

An SDK to help integrate the KILT Decentralized Identity Provider (DIP) protocol using KILT as an Identity provider.

git clone [https://github.com/XcavateBlockchain/dip-sdk-xcav](https://github.com/XcavateBlockchain/dip-sdk-xcav)

Since we are using the KILT Peregrine Provider, we have configured the Relay, Provider and Consumer rpc as shown below.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXfr_oINlL4EctTeiN80Hyp7f0i-VPcvupghlNvb1bmpZTjPJOA5lVJYk2-HSkwdI9KFxq7b1oGf46j6qyBEOyHSC4kea9IRGQTdJtrL5eSzYbZVt7Xf38IM3xHXiuumA_XaBZz-SIEKeMaV_w0qEJJpXK_g?key=HzgHVXYUZN16cG22fhmMVg" alt=""><figcaption></figcaption></figure>

A pre-funded account has been set up, so that it can fund new test accounts.

To execute the testing script and call the list\_object function you have to follow these steps:

1\. cd dip-sdk-xcav

2\. yarn install

3\. yarn test:e2e:peregrine-provider

Once you execute the third command in the terminal, you can see the balance has been transferred from the prefunded account to a newly created account, which you can see in [xcavate explorer in polkadotjs](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc-paseo.xcavate.io%3A443#/explorer).

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXexABPxH1wK3QLBeAHW_tooZhMbXe1c6ZtubdqBqYnaS9hmI2oELh_OC6jf9ta-j--S7GZdvB0vANSb7Xeu3tm8GmDaJA7f2FWrqqQfEZEkOP08atJlWXprb4icf2MbEy6Vu_SCczsltQAjEWo2f13z7t8?key=HzgHVXYUZN16cG22fhmMVg" alt=""><figcaption></figcaption></figure>

\
Before a DID can be verified it must be created. This can be done using the new account in the KILT Peregrine chain, which you can see in [KILT Peregrine explorer in polkadotjs](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fperegrine.kilt.io%2Fparachain-public-ws%2F#/explorer) .

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXes1oy0s7zwBxyMK7cxFfn1OBjyZFBcqwzPMG6M42foEoSDmHTNp0KXojrPIG21QOSGyaK69r42iD-rvDJxovvkbfgpheesfhfe2JBIxlhGyQEpm7C268bqdQBMVvLReWl-hkWUtAbchw-Yto3u5T3CaUwG?key=HzgHVXYUZN16cG22fhmMVg" alt=""><figcaption></figcaption></figure>

Once created the DID verification test can be executed. This extrinsic execution can be viewed in the [polkadot js explorer](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc-paseo.xcavate.io%3A443#/explorer).

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXdHQGmDsvfJMAHks-nSHWjc5-o9mMGleSku2D_3F06NjFjgvNTIHEBrBzdzqAzjcck2KKsKZCV0I8PJ_1SNa_0vncyhLwCXNqxnpgAAhmCjWq3cejBS7nPawhnAecb1X4Ej3uc6PkTXpihnTpLN7du27AZn?key=HzgHVXYUZN16cG22fhmMVg" alt=""><figcaption></figcaption></figure>

If the transaction is going through it means that the DID is verified. If the DID were not verified, there would be an InvalidProof error.\


\
\
