# [Gas Report]

## [G-1] Structs can be closely packed 
There are 2 instance of it 
1st instance is at [ILotterySetup.sol#L63-L78](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/ILotterySetup.sol#L63-L78) 
here in the LotterySetupParams struct `IERC20 token;` which occupies only 20 bytes which can easily pack with `uint8 selectionSize;` occupies 1 byte & `uint8 selectionMax` occupies 1 bytes by packing them we can save 1 storage slot (which saves 2100 gas)
2nd instance is at [ILotterySetup.sol#L53-L60](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/ILotterySetup.sol#L53-L60)
here in the LotteryDrawSchedule struct `uint256 firstDrawScheduledAt` is a timestamp and it won't need uint256 or 32 bytes of storage it can be used as `uint128 firstDrawScheduledAt` and `uint256 drawCoolDownPeriod` which is cool down period it will be of less duration hence we can use `uint128 drawCoolDownPeriod` and pack them to save 1 storage slot (which can save 2100 gas)

## [G-2] X += Y can be written as X = X + Y for state variables saves some gas
There are 2 instances of it 
here is the permalink of them
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L30
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L43

## [G-3]can use require instead of assert 
On failure, many functions fail with an assert which consumes all available gas of the transaction.
No gas is refunded with assert
There is 1 instance  of it 
here is the permalink of it
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L147
  