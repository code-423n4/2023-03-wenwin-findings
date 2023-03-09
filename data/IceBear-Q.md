## Non Critical Issues


### [N-1] Use of block.timestamp
Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
#### Recommended Mitigation Steps
Block timestamps should not be used for entropy or generating random numbers — i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.

Instances where block.timestamp is used:

*Find (12) instance(s) in contracts*:
```solidity
File: src/Lottery.sol

135:         if (block.timestamp < drawScheduledAt(currentDraw)) {

164:             if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {

```
[src/Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol)

```solidity
File: src/LotterySetup.sol

74:         if (initialPotDeadline < (block.timestamp + lotterySetupParams.drawSchedule.drawPeriod)) {

114:         if (block.timestamp > ticketRegistrationDeadline(drawId)) {

137:         if (block.timestamp <= initialPotDeadline) {

```
[src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)

```solidity
File: src/RNSourceController.sol

64:         if (block.timestamp - lastRequestTimestamp <= maxRequestDelay) {

70:             maxFailedAttemptsReachedAt = block.timestamp;

94:         bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;

107:         lastRequestTimestamp = block.timestamp;

```
[src/RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol)

```solidity
File: src/staking/StakedTokenLock.sol

26:         if (block.timestamp > depositDeadline) {

39:         if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {

39:         if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {

```
[src/staking/StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol)

### [N-2] Use require instead of assert
Assert should not be used except for tests, require should be used.

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

`assert() and reqire();`

The big difference between the two is that the assert()function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay.This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256). Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there’s a mistake”.

*Find (2) instance(s) in contracts*:
```solidity
File: src/LotterySetup.sol

147:         assert(initialPot > 0);

```
[src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)

```solidity
File: src/TicketUtils.sol

99:             assert((winTier <= selectionSize) && (intersection == uint256(0)));

```
[src/TicketUtils.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol)

### [N-3] For modern and more readable code; update import usages

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

#### Context
All Contracts.

#### Recommendation: 
import {contract1 , contract2} from "filename.sol";

A good example from the ArtGobblers project;
```
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```

### [N-4] fulfillRandomWords must not revert

Accordingly to [ChainLinks’ documentation](https://docs.chain.link/vrf/v2/security/#fulfillrandomwords-must-not-revert):

fulfillRandomWords must not revert If your fulfillRandomWords() implementation reverts, the VRF service will not attempt to call it a second time. Make sure your contract logic does not revert. Consider simply storing the randomness and taking more complex follow-on actions in separate contract calls made by you, your users, or an Automation Node.

*Find (1) instance(s) in contracts*:
```solidity
File: src/VRFv2RNSource.sol

32:    function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {   
33:        if (randomWords.length != 1) { 
34:            revert WrongRandomNumberCountReceived(requestId, randomWords.length);    
35:        }    
36:
37:fulfill(requestId, randomWords[0]);   
38:    }
39:}
```
[src/VRFv2RNSource.sol#L32-L39](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L32-L39)



## Low Issues


### [L-1] Empty Function Body - Consider commenting why

*Find (1) instance(s) in contracts*:
```solidity
File: src/Ticket.sol

17:     constructor() ERC721("Wenwin Lottery Ticket", "WLT") { }

```
[src/Ticket.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol)

### [L-2] Unsafe ERC20 operation(s)

*Find (3) instance(s) in contracts*:
```solidity
File: src/staking/StakedTokenLock.sol

34:         stakedToken.transferFrom(msg.sender, address(this), amount);

47:         stakedToken.transfer(msg.sender, amount);

55:         rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));

```
[src/staking/StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol)

### [L-3] Add to blacklist function

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```

#### Recommended Mitigation Steps : Add to Blacklist function and modifier.



### [S-1] Generate perfect code headers every time

I recommend using header for Solidity code layout and readability

https://github.com/transmissions11/headers


