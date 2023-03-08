## 1. Inappropriate Use of Assert in LotterySetup.sol and TicketUtils.sol

The use of assert statement on line 147 of LotterySetup.sol to verify if the initialPot variable is greater than zero is not appropriate. The purpose of assert is to check for conditions that should never happen, and using it for non-fatal errors can lead to the Panic error, which hinders debugging and results in unpredictable behavior. Therefore, to enhance the robustness of the contract and facilitate error detection and handling, custom errors (or requires, depending on the situation and design option) should be used instead. Custom errors provide better error messaging and do not terminate the contract execution.

Similarly, on line 99 of TicketUtils.sol, assert is used to check if winTier is less than or equal to selectionSize and intersection is equal to uint256(0). This usage of assert may also lead to the Panic error in non-fatal situations. It would be better to use custom errors or require statements to handle these types of errors in a more robust and transparent manner.

File: LotterySetup.sol | Line: 147 | `assert(initialPot > 0);`
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147
File: TicketUtils.sol | Line: 99 | `assert((winTier <= selectionSize) && (intersection == uint256(0)));`
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99

## 2. Use decimals() function directly from IERC20 interface instead of IERC20Metadata in LotterySetup contract.

IERC20Metadata interface to retrieve the decimals() value of the rewardToken. While this is a valid approach, it can be simplified and made more efficient by using the decimals() function directly from the IERC20 interface.

Using the decimals() function directly from the IERC20 interface avoids the need to cast the token address to IERC20Metadata, and simplifies the code. We recommend updating the code to use the decimals() function from the IERC20 interface.

File: LotterySetup.sol | Line: 79 | uint256 tokenUnit = 10 ** IERC20Metadata(address(lotterySetupParams.token)).decimals();
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L79
File: LotterySetup.sol | Line: 128 | return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L128
File: LotterySetup.sol | Line: 168 | uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L168


