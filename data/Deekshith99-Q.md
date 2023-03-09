# [QA Report]

## [NC-1] Unused imports
unused imports should be cleared for better readability of the code 
There are 5 instances of it 
here are the permalinks of them
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryToken.sol#L7
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/ILotterySetup.sol#L6
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/IRNSourceController.sol#L5
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/IReferralSystem.sol#L5
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L5

## [NC-2] Risky to use latest version (0.8.19)
Context - All contracts
latest version can be some security issues 
its recommend to use most stable version 
which can have more be secure to use 

## [NC-3] Natspec comments are missing
Context - All contracts (very few functions have)
Only in some function we have Natspec comments 
its recommend to add the Natspec comments & tags for other functions too
