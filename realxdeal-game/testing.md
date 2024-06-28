---
description: realXdeal testing with polkadot js as frontend
---

# Testing

1. Setup Game

To create a new round and initially set up the game, the setupGame function needs to be called through the SUDO account.This creates a new game setup and launches new collections for the new NFTs.

<figure><img src="https://lh7-us.googleusercontent.com/Qs0NjEuR31qgcEilRczVcWfIfGfusapwgfAlPcSaAzDCuzu_DrfkYpshfFIbCJwe4tfj9m4MhczB-wkAsMwSUXFoi26tfaz23G3_bgh0R8oaVp7Mz9bkD7enMGXgV-dtLfAdstJJFa4ICMMun_4bPPI" alt=""><figcaption></figcaption></figure>

2. Register new users

If an account has not yet been registered as a player, the SUDO account can register new users, which is initially awarded 100 points to be able to play the game.As a parameter the AccountId of the user is needed.

<figure><img src="https://lh7-us.googleusercontent.com/HHCGsdV5n10OpRIU1Nsc0QWy3j3LaoiTP2eaCup7WxO5Li0PXfQNh5BNZ98R1ZRgQUoqJKDIYhITJTjxySZxxEG58zPNu8quh51bY6cxNLEYb1hQ4blUU1HcMZW5j_MRrszt_1UIrhmAEAQnBBsGEP4" alt=""><figcaption></figcaption></figure>

3. Give points to user

The SUDO has the option to give points to users. This function will distribute 100 points to a user.As a parameter the AccountId of the user is needed.

<figure><img src="https://lh7-us.googleusercontent.com/TUyeY4C1vJ6JSJqlNS8yh1YLX0f2CTOOFKDACH3rxnTeQuskq8jfi-yHGWjl-5FAsUNx3B50-tufc53O8rpNW7gBmfpN3TpxghWFJaNv-A4Xe-_Bfz8Q5tudelHYhL-IooPD_IEGUmhZNdJ_Ph7fLZo" alt=""><figcaption></figcaption></figure>

4. Play Game

For the user / player to start a game they have to call the playGame function.As a parameter the user has to select the difficulty level. There are 3 options. Practice, Player and Pro. Each option has a different time limit and gives different rewards.

The user has to make one practice round before unlocking the other options.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

The property for the game is visible under that game info card.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

5. Submit Answer

To submit an answer the player has to call the submitAnswer function, which has the parameter of guess as u32, which is the price the player guesses and the gameId, which is the id of the players game.

The player has to make a guess within the time limit, otherwise the round is counted as a loss and the player (as long as it is not a practice round) loses some points.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

A player can see their points and collected NFTs in their account data.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

6. Trade NFTs

To trade NFTs with other players, a player can list their NFTs for trading. It is not possible for a player to transfer their NFTs to other accounts the only way to trade NFTs is via the selected functions.

To list a NFT the player has to call the listNft function. As parameters the collectionId and the itemNft of the NFT are needed.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

7. Delist NFT

The player has the option to delist their listed NFT by calling the delist function.

This function takes the parameter listingId of the listing.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

8. Make Offer

Another player can now make an offer for the listing by offering one of their NFTs. As parameters the listingId is needed and the collectionId and itemId of their NFT.

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

9. Withdraw Offer

The player who made the offer has the option to withdraw their offer, as long as the offer did not get accepted or rejected, by calling the withdrawOffer function.

This function takes the parameter offerId.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

10. Handle Offer

The owner of the listing has the option to accept the offer or reject the offer. In the first case if they reject the offer nothing happens and the offer gets deleted. In the second case if they accept the offer the two NFTs get swapped.

As a parameter this function takes the offerId and the enum with the options Accept or Reject.

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>
