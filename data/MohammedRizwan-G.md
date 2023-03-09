## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:| 
| [G&#x2011;01] | Avoid compound assignment operator in state variables | 15 |

### [G&#x2011;01]  Avoid compound assignment operator in state variables
Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).
There are 15 instances of this issue.
```solidity
File: src/Lottery.sol

129        frontendDueTicketSales[frontend] += tickets.length;
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L129)

```solidity
File: src/Lottery.sol

173            claimedAmount += claimWinningTicket(ticketIds[i]);
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L173)

```solidity
File: src/Lottery.sol

275            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L275)

```solidity
File: src/LotteryMath.sol

55        newProfit -= int256(expectedRewardsOut);
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L55)

```solidity
File: src/LotteryMath.sol

64        excessPotInt -= int256(fixedJackpotSize);
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L64)

```solidity
File: src/LotteryMath.sol

84            bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L84)

```solidity
File: src/ReferralSystem.sol

64                    totalTicketsForReferrersPerDraw[currentDraw] +=
65                        unclaimedTickets[currentDraw][referrer].referrerTicketCount;
66                }
67                totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
68            }
69            unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
70        }
71        unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
72    }
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L64-L72)

```solidity
File: src/ReferralSystem.sol

78            claimedReward += claimPerDraw(drawIds[counter]);
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L78)

```solidity
File: src/ReferralSystem.sol

147            claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L147)

```solidity
File: src/TicketUtils.sol

29                ticketSize += (ticket & uint256(1));
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L29)


```solidity
File: src/TicketUtils.sol

59            randomNumber /= currentSelectionCount;
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L59)

```solidity
File: src/TicketUtils.sol

96                winTier += uint8(intersection & uint120(1));
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L96)
