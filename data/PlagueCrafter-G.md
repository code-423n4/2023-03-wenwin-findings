# Variable not used:

**Finding:**
The `uint128 public constant DRAWS_PER_YEAR = 52;` variable is not in use

**Example(s):**
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L24

**Remediation:**
Consider removing the variable to save gas. 