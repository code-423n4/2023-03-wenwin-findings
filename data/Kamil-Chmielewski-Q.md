## QA Report

### Low-severity Issues

| # | Title | Type | Instances |
| :--: | --- | :---: | :---: |
| L-01 | Add a rounding error check in the `getPercentage` function | Arithmetics | 1 |
| L-02 | If `ticketsSoldDuringDraw < 100`, the `minimumEligibleRefferals` are calculated incorrectly | Accounting | 1 |
| Total | 2 | --- | --- |

### Non-Critical Issues

| # | Title | Type | Instances |
| :--: | --- | :---: | :---: |
| N-01 | Use named import syntax | Code Quality | All contracts in scope |
| Total | 1 | --- | --- |


### [L-01] Missing rounding error check in the `getPercentage` function

The `getPercentage` function in the `PercentageMath` library calculates the `percentage` of the `number` as follows: `number * percentage / PERCENTAGE_BASE`, where `PERCENTAGE_BASE = 100_000`. 

Because division in Solidity rounds the result down, in a situation where the `number * percentage` will be smaller than the `PERCENTAGE_BASE`, the result will be rounded down to `0`.

This situation can happen in two situations:
1. The developer forgot to multiply the `percentage` by `ONE_PERCENT` (`1000`) somewhere in the code. For example, he wanted to calculate `50%` of the number `100`. He forgot to multiply `50` by `ONE_PERCENT`. The resulting percentage will be rounded down to zero because `50 * 100 < PERCENTAGE_BASE`
2. The developer remembered to multiply by `ONE_PERCENT` everywhere in the code, but the `number` was small enough that the `number * percentage < PERCENTAGE_BASE`. If `x` is the number, and `y` is the percentage in decimal, we can easily calculate the breakpoint when this situation happens.
   $$x*1000y<100000$$
   
   $$x<{100\over y}$$
   Example: Bob wants to calculate `1%` of the number `10`. In this example, `y=1` and `x=10`. In this situation, the result will be rounded to `0` because `10<100`.

Code without the rounding error check might cause undefined behaviour. 
   
**Recommended fix**
Add either a `require` or an `if` statement in the `getPercentage` function. The following example uses a custom `RoundingError` error, which prints the `number`, `percentage` and calculation result. 
```diff
/src/PercentageMath.sol
  
library PercentageMath {
+    error RoundingError(string message, uint256 number, uint256 percentage, uint256 result);

// ...
 
 function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
+   if ((number * percentage) < PERCENTAGE_BASE && number > 0) {
+       revert RoundingError("Rounding error", number, percentage, number * percentage / PERCENTAGE_BASE);
+   }
	return number * percentage / PERCENTAGE_BASE;
 }
```

The `number > 0` check is important as well because for any percentage `y`, $0*y<100000$.

**Disclaimer**
After adding this check to the codebase, multiple unit tests are broken. They should be reviewed with care. From my experiments, most of them pass, when used with larger `number`s.

### [L-02] If `ticketsSoldDuringDraw < 100`, the `minimumEligibleRefferals` are calculated incorrectly

The `getPercentage` function (`PercentageMath.sol`) lacks the rounding error check. Because of that, the `minimumEligibleReferrals` in the `RefferalSystem` contract in certain cases are calculated incorrectly. 

In particular, the first `if` statement of the `getMinimumEligibleReferralsFactorCalculation` function in the case where `totalTicketsSoldPrevDraw` will be `<100` will round down to `0`.

```solidity
/src/ReferralSystem.sol
// ...

function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
	internal
	virtual
	returns (uint256 minimumEligible)
{
	if (totalTicketsSoldPrevDraw < 10_000) {
		// 1%
		return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT);
	}
```

As explained in the `L-01`, `x` will be rounded down to `0` when $x<{100\over y}$. In this particular case, the referral allocation is equal to `1%`. The `y=1` and as a result, $x<{100\over 1}$, where `x` is the number of `totalTicketsSoldPrevDraw`.


### [N-01] Use named import syntax
Instead of importing entire files, use named import syntax. As the repository grows, static analyzers like Slither or the Solidity compiler might raise "duplicate definition" errors because some dependencies might have overlapping names.

Additionally, using explicit import names increases readability. 

Scope: Every contract in scope. 

Example: 

```diff
diff --git a/src/Lottery.sol b/src/Lottery.sol
index 28d3477..e40adba 100644
--- a/src/Lottery.sol
+++ b/src/Lottery.sol
@@ -2,13 +2,13 @@
 // slither-disable-next-line solc-version
 pragma solidity 0.8.19;
 
-import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
-import "@openzeppelin/contracts/utils/math/Math.sol";
-import "src/ReferralSystem.sol";
-import "src/RNSourceController.sol";
-import "src/staking/Staking.sol";
-import "src/LotterySetup.sol";
-import "src/TicketUtils.sol";
+import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
+import {Math} from "@openzeppelin/contracts/utils/math/Math.sol";
+import {ReferralSystem} from "src/ReferralSystem.sol";
+import {RNSourceController} from "src/RNSourceController.sol";
+import {Staking} from "src/staking/Staking.sol";
+import {LotterySetup} from "src/LotterySetup.sol";
+import {TicketUtils} from "src/TicketUtils.sol";
```