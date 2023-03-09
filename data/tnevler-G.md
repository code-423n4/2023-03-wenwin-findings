# Report
## Gas Optimizations ##
### [G-1]: Place subtractions where the operands can't underflow in unchecked {} block
**Context:**

1. ```lotterySetupParams.drawSchedule.firstDrawScheduledAt - lotterySetupParams.drawSchedule.drawPeriod;``` [L72](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L72) 
1. ```uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;``` [L273](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L273) 

**Description:**

Some gas can be saved by using an unchecked {} block if an underflow isn't possible because of a previous require() or if-statement.

### [G-2]: Cache a value from a mapping/array in local memory
**Context:**

1. ```if (requests[requestId].status == RequestStatus.None) {``` [L34](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L34) (requests[requestId].status)
1. ```if (requests[requestId].status == RequestStatus.Fulfilled) {``` [L37](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L37) (requests[requestId].status)
1. ```if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {``` [L62](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L62) (unclaimedTickets[currentDraw][referrer].referrerTicketCount)
1. ```if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {``` [L63](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L63) (unclaimedTickets[currentDraw][referrer].referrerTicketCount)
1. ```unclaimedTickets[currentDraw][referrer].referrerTicketCount;``` [L65](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L65) (unclaimedTickets[currentDraw][referrer].referrerTicketCount)

**Description:**

If you read value from mapping/array more than once within a function then it is cheaper to cache it in local memory and then read it from memory wnen it is neaded. This will save about 40 gas.

**Recommendation:**

Example how to fix. Change:
```
if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
                if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
                    totalTicketsForReferrersPerDraw[currentDraw] +=
                        unclaimedTickets[currentDraw][referrer].referrerTicketCount;
                }
                totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
            }
            unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
```

to:

```
uint128 _referrerTicketCount = unclaimedTickets[currentDraw][referrer].referrerTicketCount;
if (_referrerTicketCount + numberOfTickets >= minimumEligible) {
                if (_referrerTicketCount < minimumEligible) {
                    totalTicketsForReferrersPerDraw[currentDraw] +=
                        _referrerTicketCount;
                }
                totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
            }
```