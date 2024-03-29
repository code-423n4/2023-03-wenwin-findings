## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

[File: LotterySetup.sol#L160-L162](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L160-L162)

```diff
-    function _baseJackpot(uint256 _initialPot) internal view returns (uint256 _fixedReward) {
+    function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {
-        return Math.min(_initialPot.getPercentage(BASE_JACKPOT_PERCENTAGE), jackpotBound);
+        _fixedReward = Math.min(_initialPot.getPercentage(BASE_JACKPOT_PERCENTAGE), jackpotBound);
    }
```
## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A `private` visibility that saves more gas on function calls than the `internal` visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier instance below may be refactored as follows:

[File: Lottery.sol#L52-L57](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L52-L57)

```diff
+    function _whenNotExecutingDraw() private view {
+        if (drawExecutionInProgress) {
+            revert DrawAlreadyInProgress();
+        }
+    }

    modifier whenNotExecutingDraw() {
-        if (drawExecutionInProgress) {
-            revert DrawAlreadyInProgress();
-        }
+        _whenNotExecutingDraw();
        _;
    }
```
## Internal function with one line of code eexcution
Internal functions entailing only one line of code may be embedded inline with the function logic invoking it.

For instance, the following code line may be refactored as follows to save gas both on function calls and contract deployment considering [`_baseJackpot()`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L160-L162) is only called once in LotterySetup.sol by `fixedReward()` and not anywhere else:

[File: LotterySetup.sol#L120-L130](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120-L130)

```diff
    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
        if (winTier == selectionSize) {
-            return _baseJackpot(initialPot);
+            return Math.min(_initialPot.getPercentage(BASE_JACKPOT_PERCENTAGE), jackpotBound);
        } else if (winTier == 0 || winTier > selectionSize) {
            return 0;
        } else {
            uint256 mask = uint256(type(uint16).max) << (winTier * 16);
            uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
            return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
        }
    }
```
## Function order affects gas consumption
The order of function will also have an impact on gas consumption. Because in smart contracts, there is a difference in the order of the functions. Each position will have an extra 22 gas. The order is dependent on method ID. So, if you rename the frequently accessed function to more early method ID, you can save gas cost. Please visit the following site for further information:

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.19",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).

As an example, the following `>=` inequality instance may be refactored as follows:

[File: LotterySetup.sol#L51](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L51)

```diff
-        if (lotterySetupParams.selectionMax >= 120) {
// Rationale for subtracting 1 on the right side of the inequality:
// x >= 10; [10, 11, 12, ...]
// x > 10 - 1 is the same as x > 9; [10, 11, 12 ...]
+        if (lotterySetupParams.selectionMax > 120 - 1) {
```
Similarly, the `<=` instance below may be replaced with `<` as follows:

[File: LotterySetup.sol#L137](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L137)

```diff
-        if (block.timestamp <= initialPotDeadline) {
// Rationale for adding 1 on the right side of the inequality:
// x <= 10; [10, 9, 8, ...]
// x < 10 + 1 is the same as x < 11; [10, 9, 8 ...]
+        if (block.timestamp < initialPotDeadline + 1) {
```
## += and -= cost more gas
`+=` and `-=` generally cost 22 more gas than writing out the assigned equation explicitly. The amount of gas wasted can be quite sizable when repeatedly operated in a loop.

For instance, the `+=` instance below may be refactored as follows:

[File: Lottery.sol#L129](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L129)

```diff
-        frontendDueTicketSales[frontend] += tickets.length;
+        frontendDueTicketSales[frontend] = frontendDueTicketSales[frontend] + tickets.length;
```
Similarly, the `-=` instance below may be refactored as follows:

[File: LotteryMath.sol#L55](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L55)

```diff
-        excessPotInt -= int256(fixedJackpotSize);
+        excessPotInt = excessPotInt - int256(fixedJackpotSize);
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the instance below may be refactored as follows:

[File: LotteryMath.sol#L83](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L83)

```diff
-        if (excessPot > 0 && ticketsSold > 0) {
+        if (!(excessPot == 0 || ticketsSold == 0)) {
```
## Use smaller uint128 or smaller type and pack structs variables to use lesser amount of storage slots
As the solidity EVM works with 32 bytes, most variables work fine with less than 32 bytes and may be packed inside a struct when sitting next to each other so that they can be stored in the same slot, this saves gas when writing to storage ~2000 gas.

For instance, struct `LotteryDrawSchedule` of ILotterySetup.sol may be refactored as follows:

[File: ILotterySetup.sol#L52-L60](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L52-L60)

```diff
/// @dev Lottery draw schedule parameters
struct LotteryDrawSchedule {
    /// @dev First draw is scheduled to take place at this timestamp
-    uint256 firstDrawScheduledAt;
+    uint128 firstDrawScheduledAt;
    /// @dev Period for running lottery
-    uint256 drawPeriod;
+    uint64 drawPeriod;
    /// @dev Cooldown period when users cannot register tickets for draw anymore
-    uint256 drawCoolDownPeriod;
+    uint64 drawCoolDownPeriod;
}
```
This alone will save ~4k gas with 3 slots of storage reduced to 1 slots.

## Unchecked SafeMath saves gas
"Checked" math, which is default in ^0.8.0 is not free. The compiler will add some overflow checks, somehow similar to those implemented by `SafeMath`. While it is reasonable to expect these checks to be less expensive than the current `SafeMath`, one should keep in mind that these checks will increase the cost of "basic math operation" that were not previously covered. This particularly concerns variable increments in for loops. When no arithmetic overflow/underflow is going to happen, `unchecked { ++i ;}` to use the previous wrapping behavior further saves gas just as in the for loop below as an example:

[File: Lottery.sol#L125-L127](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L125-L127)

```diff
-        for (uint256 i = 0; i < drawIds.length; ++i) {
+        for (uint256 i = 0; i < drawIds.length) {
            ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
+            unchecked {
+                ++i;
+            }
        }
```