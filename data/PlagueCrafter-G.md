## Duplicated if() checks should be refactored to a modifier:

**Finding:**
Multiple ` if (amount == 0) {
            revert ZeroAmountInput();
        } ` checks in Staking.sol. Lines 69-71 & 81-83. 

****Example(s):****
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L69-L81

****Remediation:****
Consider using a modifier