
| S No. | Issue | Instances |
|-----|-----|-----|
| [01] | Large multiples of ten should use scientific notation | 4
| [02] | Require() should be used instead of assert() | 2
| [03] | Adding a return statement when the function defines a named return variable, is redundant | 10
| [04] | Non-library or interface files should use fixed compiler versions, not floating ones | 3
| [05] | Use named imports instead of plain 'import file.sol' | 53
| [06] | Use a more recent version of solidity | 3

-------------

## 01 Large multiples of ten should use scientific notation

Use (e.g. 1e12) rather than decimal literals (e.g. 1000000000000), for better code readability

_There are 4 instances of this issue_

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol

```
File: src/PercentageMath.sol

8: uint256 public constant PERCENTAGE_BASE = 100_000;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
File: src/ReferralSystem.sol

117: if (totalTicketsSoldPrevDraw < 10_000) {
121: if (totalTicketsSoldPrevDraw < 100_000) {
125: if (totalTicketsSoldPrevDraw < 1_000_000) {
```

----------

## 02 Require() should be used instead of assert()

Assert should not be used except for tests, `require` should be used

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256). Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there’s a mistake”.

_There are 2 instances of this issue_

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
File: src/LotterySetup.sol

147: assert(initialPot > 0);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```
File: src/TicketUtils.sol

99: assert((winTier <= selectionSize) && (intersection == uint256(0)));
```

------------

## 03 Adding a return statement when the function defines a named return variable, is redundant

_There are 10 instances of this issue_

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
File: src/Lottery.sol

234: function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
238: function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol

```
File: src/PercentageMath.sol

17: function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
22: function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
File: src/ReferralSystem.sol

111: function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```
File: src/TicketUtils.sol

17: function isValidTicket(
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol

```
File: src/staking/Staking.sol

48: function rewardPerToken() public view override returns (uint256 _rewardPerToken) {
61: function earned(address account) public view override returns (uint256 _earned) {
```

--------

## 04 Non-library or interface files should use fixed compiler versions, not floating ones

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

_There are 3 instances of this issue_

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol

```
File: src/VRFv2RNSource.sol

3: pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol

```
File: src/interfaces/IVRFv2RNSource.sol

3: pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

```
File: src/staking/StakedTokenLock.sol

3: pragma solidity ^0.8.17;
```

-----

## 05 Use named imports instead of plain 'import file.sol'

For instance, you use regular imports such as:

[LotteryToken.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L5)

```
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

Instead of this, use named imports such as:

```
import {ERC20.sol} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

_This issue exists in all the In-scope contracts_. _There are 53 instances of this issue_

[LotteryToken.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L5-L7) , [VRFv2RNSource.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L5-L7) , [StakedTokenLock.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L5-L7) , [Staking.sol#L5-L9](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L5-L9) , [LotterySetup.sol#L5-L10](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L5-L10) , [Lottery.sol#L5-L11](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L5-L11) , [Ticket.sol#L5-L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L5-L6) , [RNSourceBase.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L5) , [RNSourceController.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L5-L7) , [ReferralSystem.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L5-L7) , [LotteryMath.sol#L5-L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L5-L6) , [IVRFv2RNSource.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L5) , [IVRFv2RNSource.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L5) , [ITicket.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ITicket.sol#L5) , [IStakedTokenLock.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStakedTokenLock.sol#L5) , [IStaking.sol#L5-L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol#L5-L6) , [IReferralSystem.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L5) , [IRNSourceController.sol#L5-L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L5-L6) , [ILottery.sol#L5-L8](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol#L5-L8) , [ILotterySetup.sol#L5-L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L5-L6)
 
--------------

## 06 Use a more recent version of solidity

_There are 3 instances of this issue_

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol

```
File: src/VRFv2RNSource.sol

3: pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol

```
File: src/interfaces/IVRFv2RNSource.sol

3: pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

```
File: src/staking/StakedTokenLock.sol

3: pragma solidity ^0.8.17;
```

-----------
