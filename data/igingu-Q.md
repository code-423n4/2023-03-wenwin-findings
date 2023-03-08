# [L-01] Ticket minting should use `safeMint` instead of `mint`

To prevent minting an NFT to an address that can not handle it, the convention is to use
[OpenZeppelin's ERC721 \_safeMint function](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L236),
which checks that receiving address can handle NFTs, as opposed to the
[discouraged \_mint function](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L264).

## Recommendation:

```solidity
function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
    ticketId = nextTicketId++;
    ticketsInfo[ticketId] = TicketInfo(drawId, combination, false);
    _mint(to, ticketId);
    ->
    _safeMint(to, ticketId);
}
```

# [L-02] `mintNativeTokens` doesn't require amount to be bigger than 0

`mintNativeTokens` allows minting 0 tokens, which is redundant, but will emit event that can be picked up by off-chain.
This should not be the case, the function should check that amount is bigger than 0.

## Recommendation:

```solidity
error MintNativeTokensAmountNotZero();

src/Lottery.sol:285:
    function mintNativeTokens(address mintTo, uint256 amount) internal override {
        if (amount == 0) {
            revert MintNativeTokensAmountNotZero();
        }
        ILotteryToken(address(nativeToken)).mint(mintTo, amount);
    }
```

# [L-03] Incomplete or mismatching documentation

Documentation misses certain important aspects, both in protocol documentation and code comments.

## ReconstructTicket function not detailed enough

[reconstructTicket function](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L43)
encapsulates a complex and complicated algorithm, comments should be more comprehensive and maybe some examples would
help.

## Mismatch with documentation for ExcessPot calculation

[ExcessPot calculation is correct but doesn't match documentation](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L63) -
[calculation documentation](https://docs.wenwin.com/wenwin-lottery/the-game/prizes/bonuses) specifies
`CumulativeNetProfit` is to be multiplied by `(1 - SafetyMargin)`,
[not SafetyMargin](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L63).

## Care should be taken about rewardsToken

Documentation should specify that the `rewardsToken` (currently DAI), should be carefully chosen - an ERC777 that, on
token transfer, calls
[ERC777TokensRecipient.tokensReceived](https://github.com/0xjac/ERC777/blob/master/contracts/examples/ReferenceToken.sol#L313),
would hand execution to msg.sender and open up multiple reentrancy vulnerabilities.

## Game rule missing from documentation but coded in the contracts

Documentation should clearly state that in case a jackpot is won, no other winTier tickets have prizes anymore. This is
not specified in the documentation or game rules but in the code, and it is confusing to the auditor and future players.
Examples of code showing that either jackpots are paid, or non-jackpots are paid:

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L50 either removes
`jackpot + potential bonus`, or `ticketsSold * expectedPayout + potential bonus`.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L208 leaves `winAmount initialized with 0` for
either jackpot or the rest of the winTiers.

## `StakedTokenLock.sol` should have more documentation towards its' purpouse and intention of use

It was hard to determine at first, what `StakedTokenLock.sol` was intended for, and multiple people had this exact
issue. There should be some comments describing the intended use of this contract, and how it will be set up after
launch.

## `packFixedRewards` should document hidden important assumptions of the function

[packFixedRewards function](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L164) should
have comments describing how the function works (it is pretty hard to figure it out), as well the assumptions without
which the function wouldn't work: the function quietly assumes that the
`individual rewards for each winTier are <= (2 ^ 16 - 1)`, otherwise one would require more than 16 bits to pack each
reward, and rewards would overlap inside the uint256.

Upon first inspection,
[the use of .decimals() -1](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L168) looks like
a bug, but is actually intended behavior, to account for the use of one decimal for ticket price (1.5 DAI) => the price
is actually stored multiplied by 10, but this is not specified in the code, and it's very confusing for the reader.

The above also goes for [unpacking](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L128).

# [L-04] `Staking.sol` should use `10 ** rewardsToken.decimals()` instead of hardcoded units amount

```solidity
src/Staking.sol:62:
    return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];
    ->
    return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / (10 ** rewardsToken.decimals()) + rewards[account];
```

# [L-05] 1 year claim deadline implementation doesn't fully match documentation

https://docs.wenwin.com/wenwin-lottery/the-game/prizes#claiming-prizes specifies "Winning tickets have a one-year
deadline **FROM THE DRAW TIME** to claim their prize by redeeming the winning NFT ticket."

[In implementation](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L164), this check is made
against the ticket registration deadline, which would be a little sooner than one-year from the ticket's draw time, by
`drawCoolDownPeriod` seconds. This is not a bug but something to be aware of.

# [L-06] One owner is a single point of failure

The `onlyOwner` role has a single point of failure and `onlyOwner` can use the critical functions
[initSource](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L77) and
[swapSource](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L89).

This poses the vulnerability that the owner's private key is lost or stolen, and hence control over the Lottery's Oracle
is lost.

## Recommendation:

Financial applications usually use multiple addresses for one role (3 owners are better than 1). An easy implementation
of this would be
[OpenZeppelin's AccessControl Roles](https://docs.openzeppelin.com/contracts/2.x/access-control#using-roles), and
setting 2 or 3 people out of the core team as owners.

# [L-07] Users should not be able to refer themselves
Users being able to refer themselves might incentivise them to act selfishly, without necessarily being aware of the Minimum Elligible Rewards mechanism. The above might lead to a lot of players with referrals, but none of them enough for actual rewards.

## Recommendation: 
[buyTickets](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L110) should revert if ```msg.sender == referrer```.

# [N-01] Documentation misses `yarn` instruction to install node modules

Project's setup misses `yarn` instruction in the README, which is problematic when one wants to run package.json scripts
like `yarn lint` or `yarn sol write`.

## Recommendation:

Add `yarn` instruction to project README's setup section.

# [N-02] Pragmas should be locked to specific compiler version

https://swcregistry.io/docs/SWC-103

Pragma statements should be allowed to float when contracts are intended for consumption by other developers, for
example with a library. Otherwise, it is expected that the developerd manually upgrades the pragmas to compile locally.

## Recommendation:

[Lock pragmas to specific compiler version](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

```solidity
src/staking/StakedTokenLock.sol:
    // SPDX-License-Identifier: UNLICENSED
    // slither-disable-next-line solc-version
    pragma solidity ^0.8.17;

// Should lock pragma or use the same pragma as Chainlink's VRFV2WrapperConsumerBase.sol
src/VRFv2RNSource.sol:
    // SPDX-License-Identifier: UNLICENSED
    // slither-disable-next-line solc-version
    pragma solidity ^0.8.7;

// Should lock pragma or use the same pragma as Chainlink's VRFV2WrapperConsumerBase.sol
src/interfaces/IVRFv2RNSource.sol
    // SPDX-License-Identifier: UNLICENSED
    // slither-disable-next-line solc-version
    pragma solidity ^0.8.7;
```

# [N-03] Pragma versions ^0.8.17 and 0.8.19 too recent to be trusted

[Most recent Solidity compiler versions](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

Risks in recent versions:

- unexpected bugs
- complex code generation features
- new language features
- exploitable vulnerabilities Unexpected bugs can be reported in recent versions; Risks related to recent releases Risks
  of complex code generation changes Risks of new language features Risks of known bugs

## Recommendation:

Switch to an older, more trusted Solidity compiler version, like 0.8.12.

# [N-04] Forgotten files from `forge init`

`script/Counter.s.sol` is leftover from initial setup of the project with forge.

## Recommendation:

Remove it.

# [N-05] Insufficient coverage

The test coverage rate of the `src` folder is only 77%. Testing all functions is best practice in terms of security
criteria.

```
| File                      | % Lines | % Functions | % Branches |
| -----------               | ------- | ----------- | ---------- |
| Lottery.sol               | 94.2 %  | 80.0 %      | 88.9 %	 |
| LotteryMath.sol           | 14.3 %  | 20.0 %      |  0.0 %	 |
| LotterySetup.sol          | 71.4 %  | 50.0 %	    | 68.8 %	 |
| PercentageMath.sol        |  0.0 %  |  0.0 %		| ---------- |
| RNSourceBase.sol          | 86.7 %  |  100 %	    | 75.0 %	 |
| RNSourceController.sol    | 94.7 %  |  100 %      | 94.4 %	 |
| ReferralSystem.sol        | 39.5 %  | 71.4 %	    | 25.0 %	 |
```

Due to its capacity, test coverage is expected to be 100%.

# [N-06] Unused return value names should removed

When explicitly returning a value, the return value's name is no longer needed

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L234
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L238

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L17
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L22

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L115

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L24

## Recommendation:

```solidity
src/Lottery.sol
    function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
    ->
    function currentRewardSize(uint8 winTier) public view override returns (uint256) {
        return drawRewardSize(currentDraw, winTier);
    }
```

Should also be changed in interfaces if applicable.

# [N-07] SolidityLang Style-Guide: Commonly used literals should be saved as constants

https://docs.soliditylang.org/en/v0.8.17/style-guide.html#constants

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L51

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L126

## Recommendation:

```solidity
src/LotterySetup.sol:39-40:
    uint8 private constant SELECTION_SIZE_MAX = 16;
    uint8 private constant SELECTION_MAX_MAX = 120;

src/LotterySetup.sol:51:
    if (lotterySetupParams.selectionMax >= SELECTION_MAX_MAX) {

src/LotterySetup.sol:126:
    uint256 mask = uint256(type(uint16).max) << (winTier * SELECTION_SIZE_MAX);
```

# [N-08] For modern and more readable code; update import usages

By importing whole Solidity files, one can end up importing more than needed, which increases gas costs and also doesn't
easily highlight the contracts/interfaces/libraries one actually needs.

All files contain such imports that need changing.

## Recommendation:

```solidity
src/Ticket.sol:6:
    import "src/interfaces/ITicket.sol";
    ->
    import { ITicket } from "src/interfaces/ITicket.sol";
```

# [N-09] Potential typos in code, or incorrectly named variables or code comments

```
src/RNSourceController.sol:19:
    uint256 private constant MAX_MAX_FAILED_ATTEMPTS = 10;
    ->
    uint256 private constant MAX_FAILED_ATTEMPTS = 10;

src/Lottery.sol:80:
    /// @param rewardsToReferrersPerDraw Percentage of native token rewards going to players.
    ->
    /// @param rewardsToReferrersPerDraw Percentage of native token rewards going to referrers.

src/LotteryMath.sol:32:
    /// @param fixedJackpotSize Fixed jackpot price
    ->
    /// @param fixedJackpotSize Fixed jackpot size

src/LotteryMath.sol:110:
    ? fixedReward + excess.getPercentage(EXCESS_BONUS_ALLOCATION)
    ->
    ? fixedJackpot + excess.getPercentage(EXCESS_BONUS_ALLOCATION) // not a bug, since in jackpot's case, fixedReward == fixedJackpot
```

# [N-10] Project should settle on a naming convention for `private/internal` functions

The common practice is that `private/internal` functions should start with an \_, or at least all follow the same
convention.

In the codebase, these functions sometimes start with an \_
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L160

and other times they don't.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L164

Should keep it consistent, probably with using underscores so that a reader can easily understand those functions can
not be used externally.

# [N-11] `notEnoughTimeReachingMaxFailedAttempts` calculation uses incorrect comparison

[This code here](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L64) makes it so that
current request is still considered active at `block.timestamp <= lastRequestTimestamp + maxRequestDelay`,

while
[calculation of notEnoughTimeReachingMaxFailedAttempts](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L94)
considers `notEnoughTimeReachingMaxFailedAttempts` to be true at
`block.timestamp < maxFailedAttemptsReachedAt (which is lastRequestTimestamp) + maxRequestDelay`. The comparison should
use <= here, to keep it consistent, although I don't see this as ever becoming a bug.

# [N-12] No need for `safeTransfer` in Staking.sol, since LOT is also a trusted token

[StakedTokenLock](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L32) states
that `safeTransferFrom` is not needed, since `stakedToken` is deployed by the protocol team and hence trusted. If we are
to stick to this, the same logic would apply to
[Staking.sol token transfers](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L73), which
are done on the `stakingToken`, also deployed by the protocol team and hence trusted.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L73
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L86

# [S-01] Use descriptive names for Contracts and Libraries

To be able to navigate the codebase more easily, abstract contracts and libraries should start with descriptive
conventions:

- abstract contract Abs\_
- library Lib\_

Applies to:

- LotteryMath.sol + PercentageMath.sol + TicketUtils.sol
- ReferralSystem.sol + RNSourceBase.sol + RNSourceController.sol + Ticket.sol
