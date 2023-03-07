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

###############  Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate ###############  

#### Impact
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

https://code4rena.com/reports/2022-12-prepo/#g-02-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-where-appropriate

#### Findings:
```
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L19
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L20
```

###############  functions with no usage ###############  

#### Impact
what is usage of requireFinishedDraw and mintNativeTokens functions in the Lottery contract. they are internal functions and i don't see anywhere you call them. if there is not usage for them so remove them to save gas.

#### Findings:
```
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L285
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L279
```