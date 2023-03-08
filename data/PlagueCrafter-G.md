# Variables not in use:

**Description of Finding:**
The following variables are not in use:

`uint256 public constant ONE_PERCENT = 1000;`
`uint128 public constant DRAWS_PER_YEAR = 52;`

**Example(s):**
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L24
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/PercentageMath.sol#L11

**Remediation:**
Consider removing the variables to save gas. 