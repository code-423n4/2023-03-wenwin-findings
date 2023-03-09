

|Sno.|Issue|Instances|Gas Savings|
|---|---|---|---|
|[G-01]|++I or I++ SHOULD BE UNCHECKED{++I} or UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS|9|360
|[G-02]|USING BOTH NAMED RETURNS AND A RETURN STATEMENT ISNT NECESSARY|11|33
|[G-03]|X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES|16|1808



### G-01 ++I or I++ SHOULD BE UNCHECKED{++I} or UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

*Number of Instances Identified: 9*

The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
125: for (uint256 i = 0; i < drawIds.length; ++i) {
172: for (uint256 i = 0; i < totalTickets; ++i) {
211: for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
169: for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
35: for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
77: for (uint256 counter = 0; counter < drawIds.length; ++counter) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```
57: for (uint256 i = 0; i < selectionSize; ++i) {
65: for (uint256 i = 0; i < selectionSize; ++i) {
68: for (uint256 j = 0; j <= currentNumber; ++j) {
```


### G-02 USING BOTH NAMED RETURNS AND A RETURN STATEMENT ISNT NECESSARY

*Number of Instances Identified: 11*

Removing unused named returns variables can reduce gas usage (MSTOREs/MLOADs) and improve code clarity. To save gas and improve code quality: consider using only one of those.
Each MLOAD and MSTORE costs `3 additional gas`

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
234: function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize)
238: function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
120: function fixedReward(uint8 winTier) public view override returns (uint256 amount)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol

```
17: function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result)
22: function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
111-115: function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
        internal
        view
        virtual
        returns (uint256 minimumEligible)
156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards)
161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```
17-24: function isValidTicket(
        uint256 ticket,
        uint8 selectionSize,
        uint8 selectionMax
    )
        internal
        pure
        returns (bool isValid)

```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol

```
48: function rewardPerToken() public view override returns (uint256 _rewardPerToken)
61: function earned(address account) public view override returns (uint256 _earned)
```


### G-03 X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES

*Number of Instances Identified: 16*

Using the addition operator instead of plus-equals saves **[113 gas]([https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
129: frontendDueTicketSales[frontend] += tickets.length;
173: claimedAmount += claimWinningTicket(ticketIds[i]);
275: currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol

```
55: newProfit -= int256(expectedRewardsOut);
64: excessPotInt -= int256(fixedJackpotSize);
84: bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
64-65: totalTicketsForReferrersPerDraw[currentDraw] +=
                        unclaimedTickets[currentDraw][referrer].referrerTicketCount;
67: totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
69: unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
71: unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
78: claimedReward += claimPerDraw(drawIds[counter]);
147: claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```
29: ticketSize += (ticket & uint256(1));
96: winTier += uint8(intersection & uint120(1));
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

```
30: depositedBalance += amount;
43: depositedBalance -= amount;
```