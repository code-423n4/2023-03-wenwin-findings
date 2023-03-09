
| S No. | Issue | Instances | Gas Savings (from provided tests) |
|-----|-----|-----|-----|
| [G-01] | x += y costs more gas than x = x + y for state variables (same applies for x -= y) | 16 | 1808 
| [G-02] | Internal functions only called once can be inlined to save gas | 10 | 400
| [G-03] | Using both named returns and a return statement isn't necessary | 10 | 30
| [G-04] | Increments can be unchecked | 9 | 360


-------------

## G-01 x += y costs more gas than x = x + y for state variables (same applies for x -= y)

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**.

_There are 16 instances of this issue_

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
File: src/Lottery.sol

129: frontendDueTicketSales[frontend] += tickets.length;
173: claimedAmount += claimWinningTicket(ticketIds[i]);
275: currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol

```
File: src/LotteryMath.sol

55: newProfit -= int256(expectedRewardsOut);
64: excessPotInt -= int256(fixedJackpotSize);
84: bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
File: src/ReferralSystem.sol

64: totalTicketsForReferrersPerDraw[currentDraw] +=
67: totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
69: unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
71: unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
78: claimedReward += claimPerDraw(drawIds[counter]);
147: claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```
File: src/TicketUtils.sol

29: ticketSize += (ticket & uint256(1));
96: winTier += uint8(intersection & uint120(1));
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

```
File: src/staking/StakedTokenLock.sol

30: depositedBalance += amount;
43: depositedBalance -= amount;
```

----------

## G-02 Internal functions only called once can be inlined to save gas

Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

_There are 10 instances of this issue_

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
File: src/LotterySetup.sol

160: function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
File: src/ReferralSystem.sol

74: function mintNativeTokens(address mintTo, uint256 amount) internal virtual;
111: function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
134: function requireFinishedDraw(uint128 drawId) internal view virtual;
156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol

```
File: src/RNSourceBase.sol

48: function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol

```
File: src/RNSourceController.sol

38: function requestRandomNumber() internal {
58: function receiveRandomNumber(uint256 randomNumber) internal virtual;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol

```
File: src/VRFv2RNSource.sol

28: function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {
```

-----

## G-03 Using both named returns and a return statement isn't necessary 

Removing unused named returns variables can reduce gas usage (MSTOREs/MLOADs) and improve code clarity. To save gas and improve code quality: consider using only one of those.

Each MLOAD and MSTORE costs **3 additional gas**

_There are 10 instances of this issue_

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
File: src/Lottery.sol

234: function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
238: function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol

```
File: src/PercentageMath.sol

17: function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
22: function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
File: src/ReferralSystem.sol

111: function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```
File: src/TicketUtils.sol

17: function isValidTicket(
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol

```
File: src/staking/Staking.sol

48: function rewardPerToken() public view override returns (uint256 _rewardPerToken) {
61: function earned(address account) public view override returns (uint256 _earned) {
```

-------

## G-04 Increments can be unchecked

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline

Prior to Solidity 0.8.0, arithmetic operations would always wrap in case of under- or overflow leading to widespread use of libraries that introduce additional checks.

Since Solidity 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

To obtain the previous behaviour, an unchecked block can be used. It saves **30-40 gas** per loop.

_There are 9 instances of this issue_

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
File: src/Lottery.sol

125: for (uint256 i = 0; i < drawIds.length; ++i) {
172: for (uint256 i = 0; i < totalTickets; ++i) {
211: for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
File: src/LotterySetup.sol

169: for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
File: src/ReferralSystem.sol

35: for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
77: for (uint256 counter = 0; counter < drawIds.length; ++counter) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```
File: src/TicketUtils.sol

57: for (uint256 i = 0; i < selectionSize; ++i)
65: for (uint256 i = 0; i < selectionSize; ++i) {
68: for (uint256 j = 0; j <= currentNumber; ++j) {
```

---
