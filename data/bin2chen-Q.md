L-01:
Ticket.mint() is recommended to use _safeMint() to avoid the receiver is contract, can not properly handle the NFT, resulting in the loss of NFT

```solidity

    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
        ticketId = nextTicketId++;
        ticketsInfo[ticketId] = TicketInfo(drawId, combination, false);
-       _mint(to, ticketId);
+       _safeMint(to, ticketId);
    }
```

L-02:claimable() It is more reasonable to use drawScheduledAt to determine the expiration, currently using ticketRegistrationDeadline () will be less drawCoolDownPeriod, but this is the purchase of cooldown, should not be counted as a draw cycle

```
    function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier) {
        TicketInfo memory ticketInfo = ticketsInfo[ticketId];
        if (!ticketInfo.claimed) {
            uint120 _winningTicket = winningTicket[ticketInfo.drawId];
            winTier = TicketUtils.ticketWinTier(ticketInfo.combination, _winningTicket, selectionSize, selectionMax);
-           if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {            
+           if (block.timestamp <= drawScheduledAt(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {
                claimableAmount = winAmount[ticketInfo.drawId][winTier];
            }
        }
    }
```
