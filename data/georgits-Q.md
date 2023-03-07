## Remove unused imports
ILotterySetup.sol - [6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L6)

IReferralSystem.sol - [5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L5)

IRNSourceController.sol - [5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L5)

LotteryToken.sol - [7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L7)

Lottery.sol - [6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L6)

## Use the latest solidity version with a stable pragma statement
[VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L3), [IVRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L3), [StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L3)


## It is better to use `_safeMint` over `_mint`(but add a `nonReentrant` modifier, since calls to `_safeMint` can reenter)
Ticket.sol - [26](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L26)

## Missing zero address checks
RNSourceBase.sol - [11](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L11)

StakedTokenLock.sol - [16](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L18)

## Misleading comment
In [RNSourceBase.sol(line 32)](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L32) it is specified that `fulfill()` must be called in the deriving contract's `requestRandomnessFromUnderlyingSource` function but then in [VRFv2RNSource](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L37)(that inherits from `RNSourceBase`) there is no call to `fulfill()` in `requestRandomnessFromUnderlyingSource()`. It is actually used in `fulfillRandomWords()`.

## There is no need of `if else` pattern
LotterySetup.sol - [fixedReward()](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L121-L129)
```
if (winTier == selectionSize) {
    return _baseJackpot(initialPot);
} else if (winTier == 0 || winTier > selectionSize) {
    return 0;
} else {
   uint256 mask = uint256(type(uint16).max) << (winTier * 16);
   uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
   return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
}
```
Refactored version:
```
if (winTier == selectionSize) {
    return _baseJackpot(initialPot);
}

if (winTier == 0 || winTier > selectionSize) {
    return 0;
}

uint256 mask = uint256(type(uint16).max) << (winTier * 16);
uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
```

## Players should not be able to claim winning tickets until the draw is finalised
`requireFinishedDraw()` must be called inside `claimWinningTicket()` to prevent users from claiming winning tickets before the draw is finalised.