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

## 3  Use constants instead of immutable variables

It is recommended to define variables as constants if they are not defined at contract creation. In certain cases, it may be beneficial to move immutable variables to the constructor, as shown in this example: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L11. Additionally, any immutable variable established at contract creation within a constructor should be written in uppercase letters for better readability.

File: Lottery.sol | Line: 29 | address public immutable override stakingRewardRecipient;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L29
File: LotterySetup.sol | Line: 15 | uint256 public immutable override minInitialPot;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L15
File: LotterySetup.sol | Line: 16 | uint256 public immutable override jackpotBound;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L16
File: LotterySetup.sol | Line: 18 | IERC20 public immutable override rewardToken;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L18
File: LotterySetup.sol | Line: 19 | IERC20 public immutable override nativeToken;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L19
File: LotterySetup.sol | Line: 21 | uint256 public immutable override ticketPrice;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L21
File: LotterySetup.sol | Line: 25 | uint256 public immutable override initialPotDeadline;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L25
File: LotterySetup.sol | Line: 26 | uint256 internal immutable firstDrawSchedule;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L26
File: LotterySetup.sol | Line: 27 | uint256 public immutable override drawPeriod;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L27
File: LotterySetup.sol | Line: 28 | uint256 public immutable override drawCoolDownPeriod;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L28
File: LotterySetup.sol | Line: 30 | uint8 public immutable override selectionSize;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L30
File: LotterySetup.sol | Line: 31 | uint8 public immutable override selectionMax;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L31
File: LotterySetup.sol | Line: 32 | uint256 public immutable override expectedPayout;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L32
File: LotterySetup.sol | Line: 34 | uint256 private immutable nonJackpotFixedRewards;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L34
File: LotteryToken.sol | Line: 14 | address public immutable override owner;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L14
File: ReferralSystem.sol | Line: 12 | uint256 public immutable override playerRewardFirstDraw;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L12
File: ReferralSystem.sol | Line: 13 | uint256 public immutable override playerRewardDecreasePerDraw;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L13
File: RNSourceBase.sol | Line: 8 | address internal immutable authorizedConsumer;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L8
File: RNSourceController.sol | Line: 17 | uint256 public immutable override maxFailedAttempts;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L17
File: RNSourceController.sol | Line: 18 | uint256 public immutable override maxRequestDelay;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L18
File: StakedTokenLock.sol | Line: 10 | IStaking public immutable override stakedToken;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L10
File: StakedTokenLock.sol | Line: 11 | IERC20 public immutable override rewardsToken;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L11
File: StakedTokenLock.sol | Line: 12 | uint256 public immutable override depositDeadline;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L12
File: StakedTokenLock.sol | Line: 13 | uint256 public immutable override lockDuration;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L13
File: Staking.sol | Line: 14 | ILottery public immutable override lottery;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L14
File: Staking.sol | Line: 15 | IERC20 public immutable override rewardsToken;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L15
File: Staking.sol | Line: 16 | IERC20 public immutable override stakingToken;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L16
File: VRFv2RNSource.sol | Line: 10 | uint16 public immutable override requestConfirmations;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L10
File: VRFv2RNSource.sol | Line: 11 | uint32 public immutable override callbackGasLimit;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L11



