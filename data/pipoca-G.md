## 1. Cache array length to improve gas efficiency in for loop.
The current implementation uses a for loop that loads the length of the array during each iteration, which can lead to unnecessary gas consumption. Therefore, we recommend caching the length of the array before entering the loop if the length of the array does not change inside the loop. Additionally, there is no need to assign an initial value of 0 as this adds extra gas.

Here is an example of how this optimization could be implemented:

`uint length = arr.length;`

`for (uint i; i < length; ++i) {
    //Operations not affecting the length of the array.
}`

By caching the length of the array outside of the loop and removing the initial value assignment, we can improve gas efficiency and reduce the cost of execution.

File: Lottery.sol | Line: 125 | for (uint256 i = 0; i < drawIds.length; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L125
File: ReferralSystem.sol | Line: 35 | for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L35
File: ReferralSystem.sol | Line: 77 | for (uint256 counter = 0; counter < drawIds.length; ++counter) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L77

##  2. Use unchecked math when appropriate 
++i should be unchecked{++i}/ when it is not possible for them to overflow, as is the case when used in for- and while-loops

File: Lottery.sol | Line: 125 | for (uint256 i = 0; i < drawIds.length; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L125
File: Lottery.sol | Line: 172 | for (uint256 i = 0; i < totalTickets; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L172
File: ReferralSystem.sol | Line: 35 | for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L35
File: TicketUtils.sol | Line: 28 | for (uint8 i = 0; i < selectionMax; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L28
File: TicketUtils.sol | Line: 57 | for (uint256 i = 0; i < selectionSize; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L57
File: TicketUtils.sol | Line: 65 | for (uint256 i = 0; i < selectionSize; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L65
File: TicketUtils.sol | Line: 95 | for (uint8 i = 0; i < selectionMax; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L95

## 3. Bitwise operations can be more gas-efficient than multiplication and division

in cases where gas savings are a high priority and the use of bitwise operations can achieve the same outcome as multiplication and division, it may be a reasonable choice for developers to use bitwise operations. However, it is important to ensure that the benefits of gas savings are balanced against the potential drawbacks, such as reduced code readability and increased risk of errors, and that any use of bitwise operations is thoroughly tested and audited to ensure correctness and efficiency.


File: Lottery.sol | Line: 130 | rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L130
File: Lottery.sol | Line: 275 | currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275
File: LotteryMath.sol | Line: 14 | uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L14
File: LotteryMath.sol | Line: 16 | uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L16
File: LotteryMath.sol | Line: 20 | uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L20
File: LotteryMath.sol | Line: 22 | uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L22
File: LotteryMath.sol | Line: 47 | uint256 ticketsSalesToPot = (ticketsSold * ticketPrice).getPercentage(TICKET_PRICE_TO_POT);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L47
File: LotteryMath.sol | Line: 53 | * ticketsSold * expectedPayout;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L53
File: LotteryMath.sol | Line: 84 | bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L84
File: LotteryMath.sol | Line: 129 | dueRewards = (ticketsSold * ticketPrice).getPercentage(rewardPercentage);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L129
File: LotterySetup.sol | Line: 79 | uint256 tokenUnit = 10 ** IERC20Metadata(address(lotterySetupParams.token)).decimals();
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L79
File: LotterySetup.sol | Line: 80 | minInitialPot = 4 * tokenUnit;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L80
File: LotterySetup.sol | Line: 81 | jackpotBound = 2_000_000 * tokenUnit;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L81
File: LotterySetup.sol | Line: 126 | uint256 mask = uint256(type(uint16).max) << (winTier * 16);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L126
File: LotterySetup.sol | Line: 127 | uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L127
File: LotterySetup.sol | Line: 128 | return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L128
File: LotterySetup.sol | Line: 153 | time = firstDrawSchedule + (drawId * drawPeriod);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L153
File: LotterySetup.sol | Line: 168 | uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L168
File: LotterySetup.sol | Line: 174 | packed |= uint256(reward) << (winTier * 16);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L174
File: PercentageMath.sol | Line: 18 | return number * percentage / PERCENTAGE_BASE;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L18
File: PercentageMath.sol | Line: 23 | return number * int256(percentage) / int256(PERCENTAGE_BASE);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L23
File: ReferralSystem.sol | Line: 123 | return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L123
File: ReferralSystem.sol | Line: 127 | return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L127
File: ReferralSystem.sol | Line: 141 | claimedReward = referrerRewardPerDrawForOneTicket[drawId] * _unclaimedTickets.referrerTicketCount;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L141
File: ReferralSystem.sol | Line: 147 | claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L147
File: ReferralSystem.sol | Line: 157 | uint256 decrease = uint256(drawId) * playerRewardDecreasePerDraw;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L157
File: Staking.sol | Line: 58 | return rewardPerTokenStored + (unclaimedRewards * 1e18 / _totalSupply);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L58
File: Staking.sol | Line: 62 | return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L62

##  4. Use of "!=" instead of ">" for unsigned integers

The use of  "!=" instead of ">" for unsigned integers can result in gas savings. 

File: Lottery.sol | Line: 208 | if (jackpotWinners > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L208
File: Lottery.sol | Line: 220 | jackpotWinners > 0,
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L220
File: LotteryMath.sol | Line: 65 | excessPot = excessPotInt > 0 ? uint256(excessPotInt) : 0;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L65
File: LotteryMath.sol | Line: 83 | if (excessPot > 0 && ticketsSold > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L83
File: LotterySetup.sol | Line: 133 | if (initialPot > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L133
File: LotterySetup.sol | Line: 147 | assert(initialPot > 0);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147
File: ReferralSystem.sol | Line: 98 | if (totalTicketsForReferrersPerCurrentDraw > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L98
File: ReferralSystem.sol | Line: 104 | if (playerRewardForDraw > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L104
File: ReferralSystem.sol | Line: 146 | if (_unclaimedTickets.playerTicketCount > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L146
File: ReferralSystem.sol | Line: 151 | if (claimedReward > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L151
File: Staking.sol | Line: 94 | if (reward > 0) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L94

## 5. x = x + y is cheaper than x += y

The use of x = x + y instead of x += y can result in gas savings, particularly when working with larger variables or when the operation is performed multiple times. This is because the Solidity compiler can optimize the code to reduce the number of operations and save gas costs.

File: Lottery.sol | Line: 129 | frontendDueTicketSales[frontend] += tickets.length;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L129
File: Lottery.sol | Line: 173 | claimedAmount += claimWinningTicket(ticketIds[i]);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L173
File: Lottery.sol | Line: 275 | currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275
File: LotteryMath.sol | Line: 84 | bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L84
File: ReferralSystem.sol | Line: 64 | totalTicketsForReferrersPerDraw[currentDraw] +=
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L64
File: ReferralSystem.sol | Line: 67 | totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L67
File: ReferralSystem.sol | Line: 69 | unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L69
File: ReferralSystem.sol | Line: 71 | unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L71
File: ReferralSystem.sol | Line: 78 | claimedReward += claimPerDraw(drawIds[counter]);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L78
File: ReferralSystem.sol | Line: 147 | claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L147
File: StakedTokenLock.sol | Line: 30 | depositedBalance += amount;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30
File: TicketUtils.sol | Line: 29 | ticketSize += (ticket & uint256(1));
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L29
File: TicketUtils.sol | Line: 96 | winTier += uint8(intersection & uint120(1));
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L96

## 6. Use of "int" Data Type for Variables that Should Never Be Negative

The use of "int" data type for variables that should never be negative can result in unnecessary memory usage and potential errors. This is because "int" data type reserves memory for both positive and negative values, even when the variable should never be negative. Additionally, using "int" for variables that should never be negative can lead to potential errors if negative values are inadvertently assigned to these variables.

File: Lottery.sol | Line: 40 | int256 public override currentNetProfit;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L40
File: Lottery.sol | Line: 275 | currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275
File: LotteryMath.sol | Line: 11 | using PercentageMath for int256;
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L11
File: LotteryMath.sol | Line: 36 | int256 oldProfit,
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L36
File: LotteryMath.sol | Line: 48 | newProfit = oldProfit + int256(ticketsSalesToPot);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L48
File: LotteryMath.sol | Line: 55 | newProfit -= int256(expectedRewardsOut);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L55
File: LotteryMath.sol | Line: 62 | function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L62
File: LotteryMath.sol | Line: 63 | int256 excessPotInt = netProfit.getPercentageInt(SAFETY_MARGIN);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L63
File: LotteryMath.sol | Line: 64 | excessPotInt -= int256(fixedJackpotSize);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L64
File: LotteryMath.sol | Line: 97 | int256 netProfit,
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L97
File: PercentageMath.sol | Line: 22 | function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L22
File: PercentageMath.sol | Line: 23 | return number * int256(percentage) / int256(PERCENTAGE_BASE);
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L23
















