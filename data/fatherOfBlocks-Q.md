**src/interfaces/ILotterySetup.sol**
- L6 - ITicket is imported, but it is never used, therefore it should be removed.


**src/interfaces/IReferralSystem.sol**
- L5 - ILotteryToken is imported, but it is never used, so it should be removed.


**src/interfaces/IReferralSystemDynamic.sol**
- L30 - The IReferralSystemDynamic interface is never used by any contract, therefore it should be removed.

- L24 - The MinimumReferralsRequirement struct is never used by any contract, therefore it should be removed.

- L6/9/14 - The errors created are never used by any contract, therefore they should be eliminated. Same as the ReferralRequirementFactorType enum.

**src/LotterySetup.sol**
- L10 - Ticket is imported, but it is never used, therefore it should be deleted.


**src/Lottery.sol**
- L6 - Math is imported, but never used, so it should be removed.


**src/LotteryMath.sol**
- L84 - A division is performed by the operation (ticketsSold * expectedPayout) and it is validated that ticketsSold > 0, but it is not validated that expectedPayout > 0.
So if expectedPayout == 0, that would raise an unhandled exception. This should be checked and the exception handled.


**src/staking/Staking.sol**
- The path src/staking/Staking.sol is somewhat confusing, since staking/staking remains, therefore the staking folder should have another name that contains the staking.sol inside.

- L67/73 - In the stake() function a stakingToken transfer is made, but the check effect interact pattern is not respected. It would be advisable to follow this pattern.


**src/staking/StakedTokenLock.sol**
- L3 - Pragma float This contracts in scope are floating the pragma version.


**src/LotteryToken.sol**
- L7 - LotteryMath is imported, but it is never used, so it should be removed.


**src/VRFv2RNSource.sol**
- L3 - Pragma float This contracts in scope are floating the pragma version.
