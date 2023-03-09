# QA Report
## Finding Summary

[**Low Severity**](#Low-Severity)
1. [**assert Used Over require**](#1-assert-Used-Over-require)
2. [**Unchecked Cast May Overflow**](#2-Unchecked-Cast-May-Overflow)
3. [**Early Use of Compiler Version**](#3-Early-Use-of-Compiler-Version)
4. [**Lack of Events For Critical Functions**](#4-Lack-of-Events-For-Critical-Functions)
5. [**fulfillRandomWords Reverts**](#5-fulfillRandomWords-Reverts)
6. [**ERC20 Decimals Assumption**](#6-ERC20-Decimals-Assumption)

[**Non-Critical**](#Non-Critical)
1. [**Underscore Notation Not Used or Not Used Consistently**](#1-Underscore-Notation-Not-Used-or-Not-Used-Consistently)
2. [**Power of Ten Literal Not In Scientific Notation**](#2-Power-of-Ten-Literal-Not-In-Scientific-Notation)
3. [**Inconsistent Named Returns**](#3-Inconsistent-Named-Returns)
4. [**Lack of NatSpec Contract Headers**](#4-Lack-of-NatSpec-Contract-Headers)
5. [**Named Returns Not Returning Named Variable**](#5-Named-Returns-Not-Returning-Named-Variable)
6. [**Magic Numbers Used**](#6-Magic-Numbers-Used)
7. [**Inconsistent Comment Capitalization**](#7-Inconsistent-Comment-Capitalization)
8. [**Inconsistent Trailing Period**](#8-Inconsistent-Trailing-Period)
9. [**Unnecessary Headers**](#9-Unnecessary-Headers)
10. [**ERC20 Token Recovery**](#10-ERC20-Token-Recovery)

[**Style Guide Violations**](#Style-Guide-Violations)
1. [**Order of Functions**](#1-Order-of-Functions)
2. [**Order of Layout**](#2-Order-of-Layout)

# Low Severity

## 1. assert Used Over require

`assert` should only be used in tests. Consider changing all occurrences of `assert` to `require`. Prior to Solidity 0.8 `require` will refund all remaining gas whereas `assert` will not. Even after Solidity 0.8 [`assert` will result in a panic](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#panic-via-assert-and-error-via-require) which should not occur in production code. As stated in the Solidity Documentation: 
> Properly functioning code should never create a Panic.

*/src/LotterySetup.sol*

```solidity
147:	assert(initialPot > 0);
```

*/src/TicketUtils.sol*

```solidity
99:	assert((winTier <= selectionSize) && (intersection == uint256(0)));
```

## 2. Unchecked Cast May Overflow

As of Solidity 0.8 overflows are handled automatically; however, not for casting. For example `uint32(4294967300)` will result in `4` without reversion. Consider using OpenZepplin's SafeCast library.

*/src/LotterySetup.sol*

```solidity
170:	uint16 reward = uint16(rewards[winTier] / divisor);
```

*/src/Lottery.sol*

```solidity
275:	currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

*/src/ReferralSystem.sol*

```solidity
69:	unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
71:	unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
```

*/src/PercentageMath.sol*

```solidity
23:	return number * int256(percentage) / int256(PERCENTAGE_BASE);
```

*/src/TicketUtils.sol*

```solidity
58:	numbers[i] = uint8(randomNumber % currentSelectionCount);
96:	winTier += uint8(intersection & uint120(1));
```

*/src/LotteryMath.sol*

```solidity
48:	newProfit = oldProfit + int256(ticketsSalesToPot);
55:	newProfit -= int256(expectedRewardsOut);
64:	excessPotInt -= int256(fixedJackpotSize);
```

## 3. Early Use of Compiler Version

Solidity `0.8.19` is used in all contracts except for [VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol), [StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol), and [IVRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol). Solidity `0.8.19` as of the start of the contest, is less than [2 weeks old](https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/).

By using a compiler version early you are volunteering as guinea pigs for potential bugs when it is not necessary. As an example in [Solidity 0.8.13](https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/), [a compiler bug](https://github.com/ethereum/solidity-blog/blob/499ab8abc19391be7b7b34f88953a067029a5b45/_posts/2022-06-15-inline-assembly-memory-side-effects-bug.md) was found 81 days after the release announcement (see links).

Consider downgrading contracts of version less `0.8.19` to a more battle-tested Solidity version.

## 4. Lack of Events For Critical Functions

Although `Transfer` events are emitted, the following functions should have explicit events as they offer transparency to the user: 
* [`deposit`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L24), [`withdraw`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L37), [`getReward`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L50).

A good example of event emission of a critical function can be seen [here](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L76).

## 5. fulfillRandomWords Reverts

The [chainlink documentation](https://docs.chain.link/vrf/v2/security/#fulfillrandomwords-must-not-revert) states: 

> If your fulfillRandomWords() implementation reverts, the VRF service will not attempt to call it a second time. Make sure your contract logic does not revert. Consider simply storing the randomness and taking more complex follow-on actions in separate contract calls made by you, your users, or an Automation Node.

The current implementation of `fulfillRandomWords` can revert, seen below:

```solidity
32:    function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {
33:        if (randomWords.length != 1) {
34:             revert WrongRandomNumberCountReceived(requestId, randomWords.length);
35:        }
36:	
37:        fulfill(requestId, randomWords[0]);
38:    }
```

## 6. ERC20 Decimals Assumption

[EIP20](https://eips.ethereum.org/EIPS/eip-20#decimals) states in regard to the `decimals` method:

> OPTIONAL - This method can be used to improve usability, but interfaces and other contracts MUST NOT expect these values to be present. 

The protocol assumes the `decimals` method is present which may affect the system if whole tokens are used.

*/src/LotterySetup.sol*

```solidity
79:	uint256 tokenUnit = 10 ** IERC20Metadata(address(lotterySetupParams.token)).decimals();
128:	return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
168:	uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1);
```

## Non-Critical

## 1. Underscore Notation Not Used or Not Used Consistently

Consider using underscore notation to help with contract readability (Ex. `23453` -> `23_453`).

*/src/ReferralSystem.sol*

```solidity
129:	return 5000;
```

## 2. Power of Ten Literal Not In Scientific Notation

Power of ten literals > `1e2` are easier to read when expressed in scientific notation. Consider expressing large powers of ten in scientific notation.

*/src/PercentageMath.sol*

```solidity
11:	uint256 public constant ONE_PERCENT = 1000;
```

## 3. Inconsistent Named Returns

All functions in the code based use named returns except for [`_baseJackpot`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L160) in *LotterySetup.sol*. Consider using a named return in order to maintain code style.

## 4. Lack of NatSpec Contract Headers

No contracts in scope have a fully annotated NatSpec contract header (`@title`, `@author`, etc. see [here](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html) for reference). Contracts have a partial NatSpec header like [this](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L9) or no header like [this](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L8). Consider adding properly annotated NatSpec headers.

## 5. Named Returns Not Returning Named Variable

Using named returns may help developers understand what is being returned, but this should be in a `@return` tag. There is no need to confuse developers. The following functions have named returns, but still return a value.

*/src/staking/Staking.sol*

* [`rewardPerToken`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L48), [`earned`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L61).

*/src/LotterySetup.sol*

* [`fixedReward`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120).

*/src/Lottery.sol*

* [`currentRewardSize`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L234), [`drawRewardSize`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L238).

*/src/ReferralSystem.sol*

* [`getMinimumEligibleReferralsFactorCalculation`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L111), [`playerRewardsPerDraw`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L156), [`referrerRewardsPerDraw`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L161).

*/src/PercentageMath.sol*

* [`getPercentage`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L17), [`getPercentageInt`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L22).

*/src/TicketUtils.sol*

* [`isValidTicket`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L17).

## 6. Magic Numbers Used

When setting up a lottery [LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol) there are a couple of magic numbers used in comparison. It is best practice to replace all magic numbers with constants.

```solidity
51:	if (lotterySetupParams.selectionMax >= 120) {
```

```solidity
60:	if (
61:	    lotterySetupParams.selectionSize > 16 || lotterySetupParams.selectionSize >= lotterySetupParams.selectionMax
62:	) {
```

[Here](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L30) is a good use of constants in the codebase.

## 7. Inconsistent Comment Capitalization

There is an inconsistency in the capitalization of comments. For example, [this line](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L146) is not capitilized while [this line](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L45) is. Consider capitilizing all comments.

## 8. Inconsistent Trailing Period

Some comments like [this](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L45) have a trailing `.` but other lines like [this](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L68) do not.

## 9. Unnecessary Headers

[Staking.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol) is the only contract in scope that has headers seen [here](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L46) and [here](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L65). The positioning of view functions or mutative functions can be inferred by the order of layout of the contract (if the contract follows the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html)). Consider removing the headers in [Staking.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol).

## 10. ERC20 Token Recovery

Consider adding a recovery function for Tokens that are accidently sent to the core contracts of the protocol. 

## Style Guide Violations

## 1. Order of Functions

The Solidity Style Guide suggests the following function order: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private.

The following contracts are not compliant (examples are only to prove the functions are out of order NOT a full description): 

* [Staking.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol): external functions are positioned after public functions.
* [LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol): external functions are positioned after public functions.
* [Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol): internal function is positioned after private functions.
* [RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol): external functions are positioned after internal functions.
* [ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol): external functions are positioned after internal functions.

## 2. Order of Layout

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) suggests the following contract layout order: Type declarations, State variables, Events, Modifiers, Functions.

The following contracts are not compliant (examples are only to prove the layout are out of order NOT a full description): 

* [LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol): Modifiers are positioned after Functions (constructor).
* [IRNSource.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IRNSource.sol): State is positioned after Events.
* [IStaking.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/interfaces/IStaking.sol): Events are positioned after Functions.