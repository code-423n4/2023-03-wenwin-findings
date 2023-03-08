## [G-1]  <X> += <Y> or <X>-=<Y> COSTS MORE GAS THAN <X> = <X> + <Y> OR <X>=<X> - <Y> FOR STATE VARIABLES . FOR EVERY CALL CAN SAVE 13 GAS 

FILE : 2023-03-wenwin/src/staking/StakedTokenLock.sol

    30 :  depositedBalance += amount;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L30)

    43:  depositedBalance -= amount;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L43)

FILE : 2023-03-wenwin/src/Lottery.sol

    129 :  frontendDueTicketSales[frontend] += tickets.length;

    173:   claimedAmount += claimWinningTicket(ticketIds[i]);

    275 :  currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L173)

##

### [2] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

FILE : 2023-03-wenwin/src/staking/Staking.sol

   48: function rewardPerToken() public view override returns (uint256 _rewardPerToken) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L48-L52)

FILE : 2023-03-wenwin/src/LotterySetup.sol

   120 : function fixedReward(uint8 winTier) public view override returns (uint256 amount) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L120)

##

### [G-3] ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT . This saves 30-40 gas

FILE : 2023-03-wenwin/src/staking/Staking.sol

lottery.nextTicketId() is always greater than lastUpdateTicketId so not possible to overflow 

  54: uint256 ticketsSoldSinceUpdate = lottery.nextTicketId() - lastUpdateTicketId;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L54)

##

### [G-4] Use functions instead of modifiers

Functions can have local variables. When a modifier is used, all of the variables used in the modifier are stored in memory for the entire duration of the function call, even if they are not used after the modifier. This can result in unnecessary gas costs. With a function, you can declare local variables that are only stored in memory for the duration of the function call, which can save on gas costs
Modifier  can make it harder to optimize the code for gas efficiency. Functions, on the other hand, are more straightforward and easier to optimize for gas

FILE : 2023-03-wenwin/src/Lottery.sol

    modifier requireValidTicket(uint256 ticket) {
        if (!ticket.isValidTicket(selectionSize, selectionMax)) {
            revert InvalidTicket();
        }
        _;
    }

    /// @dev Checks if we are not executing draw already.
    modifier whenNotExecutingDraw() {
        if (drawExecutionInProgress) {
            revert DrawAlreadyInProgress();
        }
        _;
    }

    /// @dev Checks if draw is being executed right now.
    modifier onlyWhenExecutingDraw() {
        if (!drawExecutionInProgress) {
            revert DrawNotInProgress();
        }
        _;
    }

    /// @dev Checks that ticket owner is caller of the function. Reverts if not called by ticket owner.
    /// @param ticketId Ticket id we are checking owner for.
    modifier onlyTicketOwner(uint256 ticketId) {
        if (ownerOf(ticketId) != msg.sender) {
            revert UnauthorizedClaim(ticketId, msg.sender);
        }
        _;
    }

 (https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L44-L74)

##

### [G-5] USE A MORE RECENT VERSION OF SOLIDITY

Using the latest version of a smart contract can help you reduce gas costs because the latest version is usually optimized for efficiency and can use fewer gas fees to execute the same function compared to an older version. In addition, the latest version may include new features that can further optimize gas usage, such as the use of newer algorithms or data structures.

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

FILE: 2023-03-wenwin/src/VRFv2RNSource.sol
 
    3 : pragma solidity ^0.8.7;

##






GAS-1	Using bools for storage incurs overhead	2
GAS-2	Cache array length outside of loop	3
GAS-3	Don't initialize variables with default value	9
GAS-4	Pre-increments and pre-decrements are cheaper than post-increments and post-decrements	4
GAS-5	Using private rather than public for constants, saves gas	9
GAS-6	Use shift Right/Left instead of division/multiplication if possible	3
GAS-7	Incrementing with a smaller type than uint256 incurs overhead	5
GAS-8	Use storage instead of memory for structs/arrays	4
GAS-9	Increments can be unchecked in for-loops	11
GAS-10	Use != 0 instead of > 0 for unsigned integer comparison	12