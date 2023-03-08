#1 Inappropriate Use of Assert in LotterySetup.sol

The assert statement on line 147 checks if the initialPot variable is greater than zero. While this check is intended to ensure the initial pot amount is correctly set, using assert is not ideal in this case. Assert statements are typically used to check for conditions that should never occur, and their use for non-fatal errors may lead to the Panic error, which hinders debugging and causes unexpected behavior.

Instead of assert, custom errors (or requires, depending on the situation and design option) should be used to enhance the contract's robustness and facilitate error detection and handling. Custom errors allow for better error messaging, and they do not terminate the contract execution. 

File: LotterySetup.sol | Line: 147 | assert(initialPot > 0);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147

