## 1. Unnecessary external call

The [`claimWinningTicket` function](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L261) in the `Lottery` contract performs a call to itself to obtain some information about the provided ticket through `this.claimable(ticketId);`. The use of `this` results in the contract performing an external call to itself, spending more gas than necessary. Consider making the `claimable` function public and using it directly (`(claimedAmount, winTier) = claimable(ticketId);`).

## 2. Make checks earlier

The `packFixedRewards` function [checks that the `rewards` for each `winTier` are always an exact multiple of the divisor](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L171-L173). This whole check could be swapped with the [previous line](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L170) so that it fails earlier and saves some gas.
