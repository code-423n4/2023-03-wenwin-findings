1) 1 - Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. May be unchecked, becouse an overflow or an underflow isn’t possible.
    
        if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) { 
            uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR; //@audit unchecked

    The operation `currentDraw - LotteryMath.DRAWS_PER_YEAR` cant't underflow, becouse of if operation `if (currentDraw >= LotteryMath.DRAWS_PER_YEAR)`. May be fixed:

        if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) { 
            unchecked {
                uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR; //@audit unchecked
            }

    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/Lottery.sol#L274
    
2) 2 - State variables should be cached in stack variables rather than re-reading them from storage. Caching of a state variable replaces each Gwarmaccess (**100 gas**) with a much cheaper stack read.

        if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {  // @audit re-read
            uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR; //@audit unchecked
            
    In this section `currentDraw` is a state variable, that may be cashed to save gas.  May be fixed:
    
        uint128 currentDrawTemp = currentDraw;
        if (currentDrawTemp >= LotteryMath.DRAWS_PER_YEAR) {  // @audit re-read
            uint128 drawId = currentDrawTemp - LotteryMath.DRAWS_PER_YEAR; //@audit unchecked
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/Lottery.sol#L273
    
3) 3 - Functions guaranteed to revert when called by normal users can be marked payable. 
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2)`, which costs an average of about **21 gas** per call to the function, in addition to the extra deployment cost.
    
        File: src/Lottery.sol:
            259: function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) 
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/Lottery.sol#L260
 
        File: src/staking/StakedTokenLock.sol:
            37: function withdraw(uint256 amount) external override onlyOwner {
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/staking/StakedTokenLock.sol#L37
            
        File: src/RNSourceController.sol:
            77: function initSource(IRNSource rnSource) external override onlyOwner {
            89: function swapSource(IRNSource newSource) external override onlyOwner {
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/RNSourceController.sol#L77
    
4) 4 - Use nested if and avoid multiple check combinations. Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

        File: src/LotteryMath.sol:
            83: if (excessPot > 0 && ticketsSold > 0) {
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/LotteryMath.sol#L83
    
        File: src/TicketUtils.sol:
            99: assert((winTier <= selectionSize) && (intersection == uint256(0)));
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/TicketUtils.sol#L99
    
        File: src/staking/StakedTokenLock.sol:
            39: if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/staking/StakedTokenLock.sol#L39
    

5) 5 - <x> += <y> costs more gas than <x> = <x> + <y> for state variables (-= too)

        File: src/Lottery.sol:
            129: frontendDueTicketSales[frontend] += tickets.length;
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/Lottery.sol#L129
    
        File: src/Lottery.sol:
            276: currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/Lottery.sol#L276
    
        File: src/ReferralSystem.sol:
            64: totalTicketsForReferrersPerDraw[currentDraw] += unclaimedTickets[currentDraw][referrer].referrerTicketCount;
            67: totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
            69: unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
            71: unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/ReferralSystem.sol#L64
            
        File: src/staking/StakedTokenLock.sol:
            30: depositedBalance += amount;
            43: depositedBalance -= amount;
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/staking/StakedTokenLock.sol#L30
    
6) 6 - Cache the mapping values rather than fetch it every time. 
    
    `unclaimedTickets[currentDraw][referrer].referrerTicketCount` should be cached
    
        File: src/RefferalSystem.sol:
            60: if (referrer != address(0)) {
                    uint256 minimumEligible = minimumEligibleReferrals[currentDraw];
                    if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
                        if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
                            totalTicketsForReferrersPerDraw[currentDraw] +=
                                unclaimedTickets[currentDraw][referrer].referrerTicketCount;
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/ReferralSystem.sol#L60
            
    `ticketsInfo[ticketId]` should be cached     
    
        File: src/Lottery.sol:
            266: unclaimedCount[ticketsInfo[ticketId].drawId][ticketsInfo[ticketId].combination]--;
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/Lottery.sol#L266
            
7) 7 - Using storage instead of memory for structs/arrays saves gas. When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

        File: src/RefferalSystem.sol:
            139: UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
            
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/ReferralSystem.sol#L139
    
        File: src/Lottery.sol:
            160: TicketInfo memory ticketInfo = ticketsInfo[ticketId];
    
    https://github.com/code-423n4/2023-03-wenwin/blob/81b04ad8c6768dc76379c84e6961f4276aef86b4/src/Lottery.sol#L160