## Lack of Zero Address checks will cause function to waste gas if 0 is passed as an argument

Located in [Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L110), the `buyTickets()` function and the `registerTickets()` function take in addresses as parameters but don't run zero address input validation. Running these functions would without these checks could waste gas if the user passes in 0 for either of the functions.

```
function buyTickets(
        uint128[] calldata drawIds,
        uint120[] calldata tickets,
        address frontend,
        address referrer
    )
        external
        override
        requireJackpotInitialized
        returns (uint256[] memory ticketIds)
    {
        if (drawIds.length != tickets.length) {
            revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
        }
        ticketIds = new uint256[](tickets.length);
        for (uint256 i = 0; i < drawIds.length) {
            ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
        }
        referralRegisterTickets(currentDraw, referrer, msg.sender, tickets.length);
        frontendDueTicketSales[frontend] += tickets.length;
        rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);
    }
```

## Remediation Steps

Add zero address input validation to both of the functions to fix the issue.

`require(frontend != address(0))`