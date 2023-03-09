# [L-01] No need to allocate memory in the stack for `winTier`

No need to allocate memory in the stack for `winTier` as its not used in the function `claimWinningTicket`

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L260-L261

# [QA-01] Description of the `calculateRewards` function is wrong

Its stated - `/// @dev Calculate frontend rewards amount for specific tickets sold`, it is not *only* calculating frontend reward, it should be `/// @dev Calculate particular rewards amount for specific tickets sold` as its calculating reward for frontend/staking.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L114

# [QA-02] @return description is missing for `ticketWinTier` function

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L78-L83

# [QA-03] @dev description for ITicket interface is wrong

It stated - `/// @dev Interface representing Ticket NTF.`, it should be `/// @dev Interface representing Ticket NFT.`

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ITicket.sol#L7