# Gas Optimizations Report for Wenwin contest
## Overview
During the audit, 2 gas issues were found.  
Total savings ~410. 

№ | Title | Instance Count | Saved
--- | --- | --- | ---
G-1 | Use unchecked blocks for subtractions where underflow is impossible | 6 | 210
G-2 | Use local variable cache instead of accessing mapping or array multiple times | 3 | 200

## Gas Optimizations Findings(2)
### G-1. Use unchecked blocks for subtractions where underflow is impossible
##### Description
In Solidity 0.8+, there’s a default overflow and underflow check on unsigned integers. When an overflow or underflow isn’t possible (after require or if-statement), some gas can be saved by using [unchecked blocks](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#checked-or-unchecked-arithmetic).
##### Instances
- [```lotterySetupParams.drawSchedule.firstDrawScheduledAt - lotterySetupParams.drawSchedule.drawPeriod;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L72) 
- [```return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L128) 
- [```uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L168) 
- [```uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L273) 
- [```return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L158) 
- [```return rewardsToReferrersPerDraw[Math.min(rewardsToReferrersPerDraw.length - 1, drawId)];```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L162) (checked in constructor)

##### Saved
This saves ~35.  
So, ~35*6 = 210
#
### G-2. Use local variable cache instead of accessing mapping or array multiple times
##### Description
It saves gas due to not having to perform the same key’s keccak256 hash and/or offset recalculation.
##### Instances
- [```if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L62) (unclaimedTickets[currentDraw][referrer].referrerTicketCount №1)
- [```if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L63) (unclaimedTickets[currentDraw][referrer].referrerTicketCount №2)
- [```unclaimedTickets[currentDraw][referrer].referrerTicketCount;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L65) (unclaimedTickets[currentDraw][referrer].referrerTicketCount №3)

##### Saved
This saves ~100. 
So, ~100*2 = 200