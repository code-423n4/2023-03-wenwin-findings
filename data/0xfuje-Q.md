# QA Report

## Low Risk Issues List
| Number |Issue Detail|
|:--:|:-------|
|[L-01]| Anyone can buy tickets for an unexisting draw in the future
|[L-02]| No bounds on ReferralSystem.sol's constructor configuration
|[L-03]| No bounds on RNSourceController.sol's constructor configuration
|[L-04]| Front-end operator could force set referrer rewards to their own address

Total: 4 low risk issues

## Non-Critical Issues List
| Number |Issue Detail|
|:--:|:-------|
| [N-01] | In an unlikely edge case referrer rewards can be bigger than intended
| [N-02] | Use consistent compiler version
| [N-03] | More consistent variable name for MAX_REQUEST_DELAY
| [N-04] | Inaccurate comments on Lottery.sol modifiers

Total: 4 non-critical issues

## Low Risk Issues
### [L-01] Anyone can buy tickets for an unexisting draw in the future
[Lottery.sol - buyTickets() - L111](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L111)

**Impact:** Users mistakenly buying tickets (with too large drawId) leads to loss of potential ticket profit
**Description:** Users accidentally calling `Lottery.sol` `buyTicket()` with a large enough `drawId` lose their chance for their ticket draw to happen. For example an user by mistake can set `drawId` to 52000 -> (there's 52 draw in a year, that means this draw would be in 3023, not realistic to expect that). This could be mitigated by the UI/front-end but it is not specified in the docs.
**Proof of Concept**: Copy the following function in `test/Lottery.t.sol` and run `forge test --match-test testBuyTicketUnexistingDraw -vvvv`:
```solidity
function testBuyTicketUnexistingDraw() public {
	uint128 currentDraw = lottery.currentDraw();
	vm.startPrank(USER);
	rewardToken.mint(5  ether);
	rewardToken.approve(address(lottery), 5  ether);
	buyTicket(currentDraw + 520000, uint120(0x0F), address(0));
	vm.stopPrank();
}
```
**Recommended Mitigation:**  Consider setting an upper bound for drawIds in `buyTicket()` function.


### [L-02] No bounds on ReferralSystem.sol's constructor configuration
[ReferralSystem.sol - constructor() - L27-L44](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L27-L44)

**Impact**: Possible bad configuration can lead to incorrect referrer reward allocation.
**Description:** Currently there's no bounds on `ReferralSystem.sol`'s constructor config parameters: `playerRewardsFirstDraw`, `playerRewardDecreasePerDraw`, `rewardsToReferrersPerDraw`. This means in case of a bad setup configuration can lead to incorrect reward allocations and to fix it the whole system has to be redeployed.
**Recommended Mitigation:** Examine the added code complexity and gas costs for the additional safety of setting bounds on the above mentioned variables.

### [L-03] No bounds on RNSourceController.sol's constructor configuration
[RNSourceController.sol - constructor() - L26-L35](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L26-L35)

**Impact**: Accidental incorrect configuration can lead to malfunctioning in `retry()` and `swapSource` function where `maxFailedAttempts` and `maxRequestDelay` variables are used.
**Description**: `RNSourceController.sol`'s `retry()` function checks if there's enough delay between requests and if the maximum failed attempts are reached. These variables (`maxRequestDelay`, `maxFailedAttempts`) are set in the constructor and have maximum upper bounds, but no lower bounds. This means they can be set to very low amounts. In the case of `maxRequestDelay` this could mean no or almost no delay between requests.`
**Recommended Mitigation:** Consider requiring lower bounds on these variables and the added code complexity and gas costs for the additional safety.

### [L-04] Front-end operator could force set referrer rewards to their own address
**Impact:** Front-end operator disabling referrer option and collecting the referrer fee for themselves 
**Description:** by not providing any option to set a referrer address in the UI when the user buys ticket via the front-end, it can automatically set referrer param of buyTickets to their own address collecting +~3% of fee on top of their 10% front-end fee .
**Recommended Mitigation:** Examine front-end operator's front-end code before approving their UI.


## Non-Critical Issues

### [N-01] In an unlikely edge case referrer rewards can be bigger than intended
[ReferralSystem.sol - getMinimumEligibleReferralsFactorCalculation - L129](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L129)

**Description**: `ReferralSystem.sol`'s `getMinimumEligibleReferralsFactorCalculation` returns decreasing rewards up to 1 million referral sales (1% - up to 10_000; 0.75% - up to 100_000 and 0.5% for up to 1_000_000). In an unlikely scenario where there is more than 1 million tickets sales for a referrer it returns 5000 aka 5% rewards.
**Recommended Mitigation**: Change the default return value of 5000 to a lower amount.

### [N-02] Use consistent compiler version
[IVRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L3), [VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L3), [StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L3)

**Description**: All through the codebase fixed solidity compiler version `0.8.19` is used, however there is three contracts that use a different compiler version `IVRFv2RNSource.sol` and `VRFv2RNSource.sol` use floating `^0.8.7` and StakedTokenLock.sol uses floating `^0.8.17`.
**Recommended Mitigation**: Change all contract compiler versions to fixed `0.8.19`. In the future take care to have the same consistent compiler version in all contracts.

### [N-03] More consistent variable name for MAX_REQUEST_DELAY
[RNSourceController.sol - MAX_REQUEST_DELAY - L20](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L20)

**Description**: `RNSourceController` has a variable `MAX_MAX_FAILED_ATTEMPTS` that is the upper bound for `maxFailedAttempts` set in the constructor. In order to be more consistent `MAX_REQUEST_DELAY` could be renamed to `MAX_MAX_REQUEST_DELAY`
**Recommended Mitigation**: Change `MAX_REQUEST_DELAY`'s name to `MAX_MAX_REQUEST_DELAY`.

### [N-04] Inaccurate comments on Lottery.sol modifiers
[Lottery.sol - whenNotExecutingDraw() - L51](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L51)
[Lottery.sol - onlyWhenExecutingDraw() - L59](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L59) 
**Description**: `whenNotExecutingDraw()` and `onlyWhenExecutingDraw()` modifiers have comments: *@dev Checks if we are not executing draw already.* and *@dev Checks if draw is being executed right now.* This is not completely accurate since these modifiers check and revert.
**Recommended Mitigation:**: Instead of Checks: add Checks and reverts in the comments.