## Use SafeMint when minting new tickets

The `mint` function in `Tickets.sol` can be dangerous in the event that the `msg.sender` is a contract address that is not set up to handle NFTs, resulting in the tickets becoming stuck. This can result in a bad user experience.

```solidity
    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
        ticketId = nextTicketId++;
        ticketsInfo[ticketId] = TicketInfo(drawId, combination, false);
        _mint(to, ticketId);
    }
```

Recommendations
Use the _safeMint() function (already included in the imported OpenZeppelin ERC721 contract) instead of _mint().