## Unnecassary constant computation in `getMinimumEligibleReferralsFactorCalculation`
1% exists as a constant but 0.75% and 0.5% are calculated with an unnecassary multiplication and division each time the function is called.

[ReferralSystem.sol#L123](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L123)

[ReferralSystem.sol#L127](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L127)

Consider creating additional constants in [PercentageMath](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol) for 0.75% and 0.5%.
