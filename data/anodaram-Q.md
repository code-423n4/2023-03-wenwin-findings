# No `delete` in the code

## Impact
We can easily find that there is no `delete` in the code.

## Proof of Concept
In contract `Lottery`, there are several multiple mappings such as `unclaimedCount` and `winAmount`.
Since they never deleted, the space is getting bigger and bigger, so it can cause the heavy contract.

## Recommended Mitigation Steps
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L271-L277
We can add some deletion activities here. We can delete the 1 year old data.