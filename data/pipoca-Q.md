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

## 3.  Use constants instead of immutable variables, and when correctly used, uppercase variable name.

It is recommended to define variables as constants if they are not defined at contract creation. In certain cases, it may be beneficial to move immutable variables to the constructor, as shown in this example: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L11. 

Additionally, any immutable variable established at contract creation within a constructor should be written in uppercase letters for better readability (see https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L17 as an example)

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

## 4. There is a shorthand way to write an if/else statement that can improve readability and shorten the code:

It increases code readability by making the logic more concise and easier to understand.
It reduces the overall number of lines of code (SLOC), resulting in a more efficient and maintainable codebase.

File: Lottery.sol | Line: 45 | if (!ticket.isValidTicket(selectionSize, selectionMax)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L45
File: Lottery.sol | Line: 53 | if (drawExecutionInProgress) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L53
File: Lottery.sol | Line: 61 | if (!drawExecutionInProgress) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L61
File: Lottery.sol | Line: 70 | if (ownerOf(ticketId) != msg.sender) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L70
File: Lottery.sol | Line: 121 | if (drawIds.length != tickets.length) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L121
File: Lottery.sol | Line: 135 | if (block.timestamp < drawScheduledAt(currentDraw)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L135
File: Lottery.sol | Line: 161 | if (!ticketInfo.claimed) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L161
File: Lottery.sol | Line: 164 | if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L164
File: Lottery.sol | Line: 208 | if (jackpotWinners > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L208
File: Lottery.sol | Line: 250 | if (beneficiary == stakingRewardRecipient) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L250
File: Lottery.sol | Line: 262 | if (claimedAmount == 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L262
File: Lottery.sol | Line: 272 | if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L272
File: Lottery.sol | Line: 280 | if (drawId >= currentDraw) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L280
File: LotteryMath.sol | Line: 83 | if (excessPot > 0 && ticketsSold > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L83
File: LotterySetup.sol | Line: 42 | if (address(lotterySetupParams.token) == address(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L42
File: LotterySetup.sol | Line: 45 | if (lotterySetupParams.ticketPrice == uint256(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L45
File: LotterySetup.sol | Line: 48 | if (lotterySetupParams.selectionSize == 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L48
File: LotterySetup.sol | Line: 51 | if (lotterySetupParams.selectionMax >= 120) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L51
File: LotterySetup.sol | Line: 54 | if (
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L54
File: LotterySetup.sol | Line: 60 | if (
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L60
File: LotterySetup.sol | Line: 65 | if (
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L65
File: LotterySetup.sol | Line: 74 | if (initialPotDeadline < (block.timestamp + lotterySetupParams.drawSchedule.drawPeriod)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L74
File: LotterySetup.sol | Line: 106 | if (initialPot == 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L106
File: LotterySetup.sol | Line: 114 | if (block.timestamp > ticketRegistrationDeadline(drawId)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L114
File: LotterySetup.sol | Line: 121 | if (winTier == selectionSize) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L121
File: LotterySetup.sol | Line: 123 | } else if (winTier == 0 || winTier > selectionSize) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L123
File: LotterySetup.sol | Line: 133 | if (initialPot > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L133
File: LotterySetup.sol | Line: 137 | if (block.timestamp <= initialPotDeadline) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L137
File: LotterySetup.sol | Line: 141 | if (raised < minInitialPot) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L141
File: LotterySetup.sol | Line: 165 | if (rewards.length != (selectionSize) || rewards[0] != 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L165
File: LotterySetup.sol | Line: 171 | if ((rewards[winTier] % divisor) != 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L171
File: LotteryToken.sol | Line: 23 | if (msg.sender != owner) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L23
File: ReferralSystem.sol | Line: 32 | if (_rewardsToReferrersPerDraw.length == 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L32
File: ReferralSystem.sol | Line: 36 | if (_rewardsToReferrersPerDraw[i] == 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L36
File: ReferralSystem.sol | Line: 60 | if (referrer != address(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L60
File: ReferralSystem.sol | Line: 62 | if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L62
File: ReferralSystem.sol | Line: 63 | if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L63
File: ReferralSystem.sol | Line: 89 | if (ticketsSoldDuringDraw == 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L89
File: ReferralSystem.sol | Line: 98 | if (totalTicketsForReferrersPerCurrentDraw > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L98
File: ReferralSystem.sol | Line: 104 | if (playerRewardForDraw > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L104
File: ReferralSystem.sol | Line: 117 | if (totalTicketsSoldPrevDraw < 10_000) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L117
File: ReferralSystem.sol | Line: 121 | if (totalTicketsSoldPrevDraw < 100_000) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L121
File: ReferralSystem.sol | Line: 125 | if (totalTicketsSoldPrevDraw < 1_000_000) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L125
File: ReferralSystem.sol | Line: 140 | if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L140
File: ReferralSystem.sol | Line: 146 | if (_unclaimedTickets.playerTicketCount > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L146
File: ReferralSystem.sol | Line: 151 | if (claimedReward > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L151
File: RNSourceBase.sol | Line: 18 | if (authorizedConsumer != msg.sender) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L18
File: RNSourceBase.sol | Line: 23 | if (requests[requestId].status != RequestStatus.None) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L23
File: RNSourceBase.sol | Line: 34 | if (requests[requestId].status == RequestStatus.None) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L34
File: RNSourceBase.sol | Line: 37 | if (requests[requestId].status == RequestStatus.Fulfilled) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L37
File: RNSourceController.sol | Line: 27 | if (_maxFailedAttempts > MAX_MAX_FAILED_ATTEMPTS) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L27
File: RNSourceController.sol | Line: 30 | if (_maxRequestDelay > MAX_REQUEST_DELAY) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L30
File: RNSourceController.sol | Line: 39 | if (!lastRequestFulfilled) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L39
File: RNSourceController.sol | Line: 47 | if (msg.sender != address(source)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L47
File: RNSourceController.sol | Line: 61 | if (lastRequestFulfilled) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L61
File: RNSourceController.sol | Line: 64 | if (block.timestamp - lastRequestTimestamp <= maxRequestDelay) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L64
File: RNSourceController.sol | Line: 69 | if (failedAttempts == maxFailedAttempts) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L69
File: RNSourceController.sol | Line: 78 | if (address(rnSource) == address(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L78
File: RNSourceController.sol | Line: 81 | if (address(source) != address(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L81
File: RNSourceController.sol | Line: 90 | if (address(newSource) == address(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L90
File: RNSourceController.sol | Line: 95 | if (notEnoughRetryInvocations || notEnoughTimeReachingMaxFailedAttempts) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L95
File: StakedTokenLock.sol | Line: 26 | if (block.timestamp > depositDeadline) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L26
File: StakedTokenLock.sol | Line: 39 | if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L39
File: Staking.sol | Line: 31 | if (address(_lottery) == address(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L31
File: Staking.sol | Line: 34 | if (address(_rewardsToken) == address(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L34
File: Staking.sol | Line: 37 | if (address(_stakingToken) == address(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L37
File: Staking.sol | Line: 50 | if (_totalSupply == 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L50
File: Staking.sol | Line: 69 | if (amount == 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L69
File: Staking.sol | Line: 81 | if (amount == 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L81
File: Staking.sol | Line: 94 | if (reward > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L94
File: Staking.sol | Line: 109 | if (from != address(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L109
File: Staking.sol | Line: 113 | if (to != address(0)) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L113
File: TicketUtils.sol | Line: 69 | if (selected[j]) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L69
File: VRFv2RNSource.sol | Line: 33 | if (randomWords.length != 1) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L33

## 5. Best practices for initializing ERC721 token contract and emitting successful deployment event

The constructor function's code block may be left empty if the ERC721 constructor is called with the appropriate arguments. This will ensure that the ERC721 token operates correctly by performing all necessary initializations. Nonetheless, it is generally advisable to emit an event after the ERC721 constructor has been called to indicate that the contract has been successfully deployed.

File: Ticket.sol | Line: 17 | constructor() ERC721("Wenwin Lottery Ticket", "WLT") { }
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L17

## 6. Use of Underscore for Number Literals in PercentageMath.sol

In the file PercentageMath.sol, on line 11, the constant ONE_PERCENT is defined as 1000. It is recommended to use an underscore for number literals to improve readability and avoid errors. Therefore, it is recommended to define ONE_PERCENT as 10_000 instead of 1000.

Using underscores for number literals improves readability and makes it easier to understand the number of zeros in large numbers. It also helps to avoid errors that can occur due to typos or missing zeros. Therefore, using underscores is a good practice and should be followed consistently throughout the codebase.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L11


