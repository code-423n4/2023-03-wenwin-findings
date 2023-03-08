## `referralRegisterTickets` function optimization

The following code can be optimized in order to reduce the number of storage access of the same slot when the required value is not changing, by caching the value. The reference to the storage slot can when using multiple nested mappings can be cached to reduce the number of slot location calculations, as well as improve readability.

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L52-L72

## Recommendation
The optimized solution could be implemented as the following:
```solidity
    function referralRegisterTickets(
        uint128 currentDraw,
        address referrer,
        address player,
        uint256 numberOfTickets
    )
        internal
    {
        if (referrer != address(0)) {
            uint256 minimumEligible = minimumEligibleReferrals[currentDraw];
            UnclaimedTicketsData storage unclaimedReferrerData = unclaimedTickets[currentDraw][referrer];
            uint256 referrerTicketCount = unclaimedReferrerData.referrerTicketCount;

            if (referrerTicketCount + numberOfTickets >= minimumEligible) {
                if (referrerTicketCount < minimumEligible) {
                    totalTicketsForReferrersPerDraw[currentDraw] += referrerTicketCount;
                }
                totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
            }
            unclaimedReferrerData.referrerTicketCount = uint128(referrerTicketCount + numberOfTickets);
        }
        unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
    }
```

