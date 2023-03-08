| | issue |
| ----------- | ----------- |
| 1 | [state variables should be cached in stack variables rather than re-reading them from storage](#1-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage) |
| 2 | [Unused local variable](#2-unused-local-variable) |
| 3 | [Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement](#3-add-unchecked--for-subtractions-where-the-operands-cannot-underflow-because-of-a-previous-require-or-if-statement) |
| 4 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#4-x--y-costs-more-gas-than-x--x--y-for-state-variables) |
| 5 | [not using the named return variables when a function returns, wastes deployment gas](#5-not-using-the-named-return-variables-when-a-function-returns-wastes-deployment-gas) |
| 6 | [can make the variable outside the loop to save gas](#6-can-make-the-variable-outside-the-loop-to-save-gas) |
| 7 | [require() or revert() statements that check input arguments should be at the top of the function](#7-require-or-revert-statements-that-check-input-arguments-should-be-at-the-top-of-the-function) |
| 8 | [internal functions only called once can be inlined to save gas](#8-internal-functions-only-called-once-can-be-inlined-to-save-gas) |
| 9 | [public functions not called by the contract should be declared external instead](#9-public-functions-not-called-by-the-contract-should-be-declared-external-instead) |
| 10 | [Use assembly to check for address(0)](#10-use-assembly-to-check-for-address0) |
| 11 | [before some functions we should check some variables for possible gas save](#11-before-some-functions-we-should-check-some-variables-for-possible-gas-save) |
| 12 | [Non-strict inequalities are cheaper than strict ones](#12-non-strict-inequalities-are-cheaper-than-strict-ones) |
| 13 | [Setting the constructor to payable](#13-setting-the-constructor-to-payable) |
| 14 | [instead of assigning to zero use `delete`](#14-instead-of-assigning-to-zero-use-delete) |


## 1. state variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

cache `currentDraw` to save 97 gas by only risking 3 gas loss if the condition passes which is rare. cache before if statement
- [Lottery.sol#L135](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L135)

cache `currentDraw`saves 97 gas by risking 3 gas loss if the condition fails which usually doesnt
- [Lottery.sol#L272](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L272)

use the cached version here for `failedSequentialAttempts` instead of re reading from storage
- [RNSourceController.sol#L73](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L73)

`unclaimedTickets[currentDraw][referrer].referrerTicketCount` should be cached because there is high chance for high gas save as there is 3 complex reads and 2 of them can be stopped.cache it before if statement like `minimumEligible`
- [ReferralSystem.sol#L62](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L62)


## 2. Unused local variable

Some of the variables are fetched and allocated but not used which means there is stack variables being made for no reason and it wastes gas

its not used so there is no need for it to be made
- [Lottery.sol#L260](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L260)


## 3. Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
this will stop the check for overflow and underflow so it will save gas

it is checked in L67 so it cant underflow
- [LotterySetup.sol#L72](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L72)

checked in L272
- [Lottery.sol#L273](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L273)


## 4. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas (13 or 113 each dependant on the usage see [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8))

- [Lottery.sol#L129](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L129)
- [Lottery.sol#L275](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275)

- [StakedTokenLock.sol#L30](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30)
- [StakedTokenLock.sol#L43](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L43)

- [ReferralSystem.sol#L64](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L64)
- [ReferralSystem.sol#L67](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L67)
- [ReferralSystem.sol#L69](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L69)
- [ReferralSystem.sol#L71](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L71)


## 5. not using the named return variables when a function returns, wastes deployment gas

- [LotterySetup.sol#L120](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120)

- [Lottery.sol#L234](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L234)
- [Lottery.sol#L238](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L238)

- [Staking.sol#L48](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L48)
- [Staking.sol#L61](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L61)

- [ReferralSystem.sol#L115](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L115)
- [ReferralSystem.sol#L156](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L156)
- [ReferralSystem.sol#L161](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L161)


## 6. can make the variable outside the loop to save gas

make the variable outside and only give the value to variable inside

- [LotterySetup.sol#L170](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L170)


## 7. require() or revert() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

L133 if statement is the most expensive check do it after the other 2 checks which are cheaper
- [LotterySetup.sol#L133](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L133)


## 8. internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

functions below are small functions that it makes sense that they be inlined.

- [LotterySetup.sol#L160](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L160)

- [ReferralSystem.sol#L161](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L161)


## 9. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

`currentRewardSize`
- [Lottery.sol#L234](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L234)


## 10. Use assembly to check for address(0)

saves 6 gas per instance

- [LotterySetup.sol#L42](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L42)

- [Staking.sol#L31](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L31)
- [Staking.sol#L34](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L34)
- [Staking.sol#L37](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L37)
- [Staking.sol#L109](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L109)
- [Staking.sol#L113](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L113)

- [RNSourceController.sol#L78](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L78)
- [RNSourceController.sol#L81](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L81)
- [RNSourceController.sol#L90](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L90)

- [ReferralSystem.sol#L60](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L60)


## 11. before some functions we should check some variables for possible gas save

before transfer we should check for amount being 0 so the function doesnt run when its not gonna do anything

- [Lottery.sol#L156](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L156)
- [Lottery.sol#L175](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L175)

- [StakedTokenLock.sol#L47](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L47)


## 12. Non-strict inequalities are cheaper than strict ones

In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).
consider replacing >= with the strict counterpart > and add `- 1` to the second side

- [LotterySetup.sol#L51](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L51)
- [LotterySetup.sol#L56](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L56)
- [LotterySetup.sol#L61](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L61)
- [LotterySetup.sol#L66](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L66)
- [LotterySetup.sol#L137](https://github.com/code-1373n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L42)

- [Lottery.sol#L164](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L164)
- [Lottery.sol#L272](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L272)
- [Lottery.sol#L280](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L280)

- [RNSourceController.sol#L64](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L64)

- [ReferralSystem.sol#L62](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L62)
- [ReferralSystem.sol#L140](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L140)


## 13. Setting the constructor to payable

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

- [LotterySetup.sol#L](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L)
- [Lottery.sol#L](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L)
- [StakedTokenLock.sol#L](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L)
- [Staking.sol#L](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L)
- [RNSourceBase.sol#L](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L)
- [RNSourceController.sol#L](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L)
- [ReferralSystem.sol#L](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L)


## 14. instead of assigning to zero use `delete` 

assigning to zero uses more gas than using `delete` , and they both assign variable to default value. so it is encouraged if the data is no longer needed use delete instead.

- [Lottery.sol#L255](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L255)

- [Staking.sol#L95](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L95)

- [RNSourceController.sol#L52](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L52)
- [RNSourceController.sol#L53](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L53)
- [RNSourceController.sol#L99](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L99)
- [RNSourceController.sol#L100](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L100)

- [ReferralSystem.sol#L142](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L142)
- [ReferralSystem.sol#L148](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L148)

