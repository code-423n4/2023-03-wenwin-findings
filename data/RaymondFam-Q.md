## Modularity on import usages
For cleaner Solidity code in conjunction with the rule of modularity and modular programming, use named imports with curly braces instead of adopting the global import approach.

For instance, the import instances below could be refactored conforming to most other instances in the code bases as follows:

[File: LotteryToken.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L5-L7)

```diff
- import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
+ import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
- import "src/interfaces/ILotteryToken.sol";
+ import {ILotteryToken} from "src/interfaces/ILotteryToken.sol";
- import "src/LotteryMath.sol";
+ import {LotteryMath} from "src/LotteryMath.sol";
```
## Sanity checks at the constructor
Adequate zero address and zero value checks should be implemented at the constructor to avoid accidental error(s) that could result in non-functional calls associated with it particularly when assigning immutable variables.

For instance, the constructor below may be refactored as follows  :

[File: StakedTokenLock.sol#L16-L22](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L16-L22)

```solidity
+    error ZeroAmountInput();
+    error ZeroAddressInput();

    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
+        if (address(_stakedToken) == address(0)) {
+            revert ZeroAddressInput();
+        }
+        if (_depositDeadline == 0 || _lockDuration == 0) {
+            revert ZeroAmountInput();
+        }
        _transferOwnership(msg.sender);
        stakedToken = IStaking(_stakedToken);
        rewardsToken = stakedToken.rewardsToken();
        depositDeadline = _depositDeadline;
        lockDuration = _lockDuration;
    }
```
## Use `delete` to clear variables
`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address or a boolean to false. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at x.

The delete key better conveys the intention and is also more idiomatic.

For instance, the `a[x] = 0` instance below may be refactored as follows:

[File: Staking.sol#L95](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L95)

```diff
-            rewards[msg.sender] = 0;
+            delete rewards[msg.sender];
```
## Return statement on named returns

Functions with named returns should not have a return statement to avoid unnecessary confusion.

For instance, the following `fixedReward()` may be refactored as follows:

[File: LotterySetup.sol#L120-L130](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120-L130)

```diff
    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
        if (winTier == selectionSize) {
-            return _baseJackpot(initialPot);
+            amount = _baseJackpot(initialPot);
        } else if (winTier == 0 || winTier > selectionSize) {
-            return 0;
+            amount 0;
        } else {
            uint256 mask = uint256(type(uint16).max) << (winTier * 16);
            uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
-            return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
+            amount extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
        }
    }

```