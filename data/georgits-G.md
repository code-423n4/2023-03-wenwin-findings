## `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables
StakedTokenLock.sol - [30](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30), [43](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L43)

Lottery.sol - [275](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275)

## Using `delete` instead of setting values to `0` saves gas
Staking.sol - [95](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L95)

Lottery.sol - [255](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L255)

ReferralSystem.sol - [142](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L142), [148](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L148)

Lottery.sol - [160](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L160)

## Use `storage` instead of `memory` for arrays/structs to save gas
ReferralSystem.sol - [139](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L139)(also remove line [145](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L145))

## Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
Lottery.sol - [193](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L193), [194](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L194), [266](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L266)