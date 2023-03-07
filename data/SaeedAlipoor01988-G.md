###############  ++i/i++ should be unchecked{++i}/unchecked{i++} ###############  

#### Impact
++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.

use this pattern :

        for (uint256 i = 0; i < _actions.length; ) {

            unchecked {
                ++i;
            }
        }

#### Findings:
```
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L125
```