## [LOW] Lottery does not accept reward tokens with 0 decimals.
The constructor of `LotterrySetup` calls the function `packFixedRewards()` where the variable `divisor` is calculated by susbtracting 1 to the decimals of the `rewardToken`. Thus, if the reward token has 0 decimals, the operation would  be reverted as `0-1` overflows.
```solidity
File: LotterySetup.sol in packFixedRewards
L168: uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1);
```

## [QA] Pot initialization can be front-runned 
The function `finalizeInitialPotRaise()` can be called by anyone, setting the `initialPot` to any amount (>= minInitialPot)
```solidity
File: LotterySetup.sol
L132: function finalizeInitialPotRaise() external override {
```