## [G-01] Setting the constructor to payable 

Making the constructor to payable can eliminate the check for `msg.value`, saving 13 gas on deployment

*There are 10 instances of this issue*
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L17
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L13
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L27
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L26
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L17
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L41
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L11
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L22
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L16
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L84

---

## [G-02] Optimal Comparison

When comparing integers, it is cheaper to use strict > & < operators over >= & <= operators, even if you must increment or decrement one of the operands. Note: before using this technique, it's important to consider whether incrementing/decrementing one of the operators could result in an over/underflow. This optimization is applicable when the optimizer is turned off.

*There are 13 instances of this issue*
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L68
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L99
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L62
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L140
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L64
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L51
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L56
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L61
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L66
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L137
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L164
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L272
https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L280

---

## [G-03] += costs more gas than = + for state variables

Use =+/=- to save gas

*There are 13 instances of this issue*
```
TicketUtils.sol:29:                ticketSize += (ticket & uint256(1));
TicketUtils.sol:96:                winTier += uint8(intersection & uint120(1));
ReferralSystem.sol:64:                    totalTicketsForReferrersPerDraw[currentDraw] +=
ReferralSystem.sol:67:                totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
ReferralSystem.sol:69:            unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
ReferralSystem.sol:71:        unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
ReferralSystem.sol:78:            claimedReward += claimPerDraw(drawIds[counter]);
ReferralSystem.sol:147:            claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
LotteryMath.sol:84:            bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
staking/StakedTokenLock.sol:30:        depositedBalance += amount;
Lottery.sol:129:        frontendDueTicketSales[frontend] += tickets.length;
Lottery.sol:173:            claimedAmount += claimWinningTicket(ticketIds[i]);
Lottery.sol:275:            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
LotteryMath.sol:55:        newProfit -= int256(expectedRewardsOut);
LotteryMath.sol:64:        excessPotInt -= int256(fixedJackpotSize);
staking/StakedTokenLock.sol:43:        depositedBalance -= amount;
```

---
