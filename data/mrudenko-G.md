https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L61
unclaimedTickets[currentDraw][referrer] - duplicated 4 times
Better to add local variable to reduce gas cost


https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L136
_unclaimedTickets can be created with `storage` keyword and lines 143 and 149 could be rewritten as
_unclaimedTickets.referrerTicketCount = 0;
_unclaimedTickets.playerTicketCount = 0;
Line 146 could be deleted in that case
;