## Unnecessary read from storage in `claimPerDraw`

In the `claimPerDraw` function in `ReferralSystem.sol`, there is an unnecessary, second read of `unclaimedTickets[drawId][msg.sender]` from storage. 

```solidity
UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {
            claimedReward = referrerRewardPerDrawForOneTicket[drawId] * _unclaimedTickets.referrerTicketCount;
            unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
        }

        _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.playerTicketCount > 0) {
            claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
            unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
        }
```
`_unclaimedTickets.playerTicketCount` is not updated between the first and second reads from storage to memory, making the second read unneeded. 

## Redundant check in `swapSource`

In the `swapSource` function of `RNSourceController.sol` the check... 
```solidity
bool notEnoughRetryInvocations = failedSequentialAttempts < maxFailedAttempts;
```
...is not needed, because of the next line...
```solidity
bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;
```
... `maxFailedAttemptsReachedAt` is always 0 when `failedSequentialAttempts` is less than `maxFailedAttempts`

The only time the second check would not catch the case is if `block.timestamp` was less than `maxRequestDelay`, which would not be an appropriate configuration.

## Unused local variable in `claimWinningTicket`

In the `claimWinningTicket` function of `Lottery.sol`, the `winTier` variable is declared, written to, and never used.

```solidity
    function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {
        uint256 winTier;
        (claimedAmount, winTier) = this.claimable(ticketId);
        if (claimedAmount == 0) {
            revert NothingToClaim(ticketId);
        }

        unclaimedCount[ticketsInfo[ticketId].drawId][ticketsInfo[ticketId].combination]--;
        markAsClaimed(ticketId);
        emit ClaimedTicket(msg.sender, ticketId, claimedAmount);
    }
```