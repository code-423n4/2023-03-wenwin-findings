## ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

Files: 

- [src/ReferralSystem.sol#L35](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L35)

This 
```
        for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
            if (_rewardsToReferrersPerDraw[i] == 0) {
                revert ReferrerRewardsInvalid();
            }
        }
```
can be replaced with

```
       for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length;) {
            if (_rewardsToReferrersPerDraw[i] == 0) {
                revert ReferrerRewardsInvalid();
            }
       unchecked{++i}
        }
```