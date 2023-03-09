### Unnecessary Reassignment in ReferralSystem.claimPerDraw()

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L145

`_unclaimedTickets` is reassigned after `unclaimedTickets[drawId][msg.sender].referrerTicketCount` is updated to zero. But the `referrerTicketCount` field is not used after `_unclaimedTickets` is reassigned. So `_unclaimedTickets` could be used for further calculations without reassignment.