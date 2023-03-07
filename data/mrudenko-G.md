https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L61
unclaimedTickets[currentDraw][referrer] - duplicated 4 times
Better to add local variable to reduce gas cost