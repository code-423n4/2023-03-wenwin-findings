## Design Flaw In The retry() Function

The retry() function here https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L60-L74 calculates failed attempts and stores them in a variable. If the value of the failed attempts goes beyond
the `maxFailedAttempts` the owner can call the function swapSource here  https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L89 
The flaw is that user can keep calling the retry function to generate the random number from the old source 
until the owner calls the swapSource.

Consider for some reason the owner is unable to call the swapSource , then the user would not be able to generate a new fresh random number , and waits for the owner to call the swapSource to get the functionality working again.

Recommendation:

Add a check like this `require (failedAttempts <= maxFailedAttempts)`


## requestConfirmations Must Be Set Above A Minimum Threshold

According to the chainlink documentation here https://docs.chain.link/vrf/v2/security/#choose-a-safe-block-confirmation-time-which-will-vary-between-blockchains the requestConfirmation value should be greater than 
a minimum threshold value , in principle the longer we wait the safer the secure the random value is.

In principle, miners/validators of your underlying blockchain could rewrite the chain’s history to put a randomness request from your contract into a different block, which would result in a different VRF output.

Affected LOC:
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L23

## fulfillRandomWords Should Not Revert

According to the chainlink documentation https://docs.chain.link/vrf/v2/security/#fulfillrandomwords-must-not-revert the function fulfillRandomWords (https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L32) should not revert and it does in the function.
The current implementation of WenWin only intends to call it once and it does seem that this is harmless, but
if they decide to get an upgrade which includes multiple random values this can be problematic , VRF service will not attempt to call it a second time. 

## Lack Of Transaction Handling Mechanism Issue.

This issue is regarding the LotteryToken.sol
This is a very common issue, and it has already caused millions of dollars in losses for lots of token users! More details here https://docs.google.com/document/d/1Feh5sP6oQL1-1NHi-X1dbgT3ch2WdhbXRevDN681Jv4/edit

Recommendation:

Add the following code to the transfer(_to address, ...) function:

`require( _to != address(this) );`

## Owner Privilege

Currently, the total supply of the tokens is minted to the owner of the contract, and the distribution of tokens is controlled by the owner.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L19
Consider transferring the tokens initially to a multi-sig account so that the tokens are protected by multiple members during the distribution and vesting period.



