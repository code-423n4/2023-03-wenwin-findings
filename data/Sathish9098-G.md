##

### [G-1] Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

FILE : 2023-03-wenwin/src/interfaces/ILotterySetup.sol

    struct LotterySetupParams {
    
    IERC20 token; // memory 20
    
    LotteryDrawSchedule drawSchedule; // memory 32
  
    uint256 ticketPrice; //memory  32
 
    uint8 selectionSize; // memory 1
 
    uint8 selectionMax; // memory 1
   
    uint256 expectedPayout; // memory  32
 
    uint256[] fixedRewards; //as per size 
}

> Currently total 6 slots 

If we repack variable its possible to save 1 slot 

    /// @audit Variable ordering with 5 slots instead of the current 6:
    struct LotterySetupParams {
    
     IERC20 token; // memory 20

   +  uint8 selectionSize; // memory 1
 
   + uint8 selectionMax; // memory 1
    
    LotteryDrawSchedule drawSchedule; // memory 32
  
    uint256 ticketPrice; //memory  32
 
   -  uint8 selectionSize; // memory 1
 
   -  uint8 selectionMax; // memory 1
   
    uint256 expectedPayout; // memory  32
 
    uint256[] fixedRewards; //as per size 
   }

   

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

FILE : 2023-03-wenwin/src/ReferralSystem.sol

    64: totalTicketsForReferrersPerDraw[currentDraw] +=
        unclaimedTickets[currentDraw][referrer].referrerTicketCount;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L64-L65)

   67: totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L67)

   69: unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L69)

   70:  unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L71)

 

##

### [2] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

FILE : 2023-03-wenwin/src/staking/Staking.sol

   48: function rewardPerToken() public view override returns (uint256 _rewardPerToken) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L48-L52)

FILE : 2023-03-wenwin/src/LotterySetup.sol

   120 : function fixedReward(uint8 winTier) public view override returns (uint256 amount) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L120)

FILE : 2023-03-wenwin/src/ReferralSystem.sol

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L111-L115)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L156)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L161)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/PercentageMath.sol#L17-L19)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/PercentageMath.sol#L22-L24)

FILE : 2023-03-wenwin/src/TicketUtils.sol

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L17-L24)
##

### [G-3] ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT . This saves 30-40 gas

SOLUTION:

require(a < b); x = b - a => require(a <b); unchecked { x = b - a } 

require(a > b); x = a - b => require(a > b); unchecked { x = a - b } 


FILE : 2023-03-wenwin/src/Lottery.sol

  if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {
  uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;

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

### [G-6] Use assembly for address(0) checks 

FILE : 2023-03-wenwin/src/RNSourceController.sol

     78 : if (address(rnSource) == address(0)) {
     90 : if (address(newSource) == address(0)) {
     81 : if (address(source) != address(0)) {

FILE : 2023-03-wenwin/src/ReferralSystem.sol

   60 : if (referrer != address(0)) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L60)

FILE :2023-03-wenwin/src/staking/Staking.sol

    if (address(_lottery) == address(0)) {
            revert ZeroAddressInput();
        }
        if (address(_rewardsToken) == address(0)) {
            revert ZeroAddressInput();
        }
        if (address(_stakingToken) == address(0)) {
            revert ZeroAddressInput();
        }
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L31-L40)

FILE :2023-03-wenwin/src/LotterySetup.sol

    if (address(lotterySetupParams.token) == address(0)) {
            revert RewardTokenZero();
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L42-L43)



##

### [G-6] USE REQUIRE INSTEAD OF ASSERT

When a require statement fails, any gas spent after the failing condition is refunded to the caller. This can help prevent unnecessary gas usage and make the contract more efficient. On the other hand, when an assert statement fails, all remaining gas is consumed and cannot be refunded, which can be wasteful and lead to more expensive transactions

The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling. 

FILE : 2023-03-wenwin/src/TicketUtils.sol

     99: assert((winTier <= selectionSize) && (intersection == uint256(0)));

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L99)

##

### [G-7] SPLITTING ASSERT() STATEMENTS THAT USE && SAVES GAS

Instead of using the && operator in a single ASSERT statement to check multiple conditions, using multiple require statements with 1 condition per ASSERT statement will save 30 GAS per &&

FILE : 2023-03-wenwin/src/TicketUtils.sol

     99: assert((winTier <= selectionSize) && (intersection == uint256(0)));

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L99)

FILE : 2023-03-wenwin/src/LotterySetup.sol

    147: assert(initialPot > 0);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L147)

##

### [G-8] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports

FILE : 2023-03-wenwin/src/LotteryMath.sol

     83 : if (excessPot > 0 && ticketsSold > 0) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L83)

FILE : 2023-03-wenwin/src/staking/StakedTokenLock.sol

  39:  if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L39)

##

### [G-9] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations

FILE : 2023-03-wenwin/src/Lottery.sol

     26: mapping(address => uint256) private frontendDueTicketSales;
     27: mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L26-L27)

     36: mapping(uint128 => uint120) public override winningTicket;
     37: mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
     39: mapping(uint128 => uint256) public override ticketsSold;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L36-L37)

##

### [G-10] Duplicated require()/if() checks should be refactored to a modifier or function

FILE : 2023-03-wenwin/src/staking/Staking.sol

    69: if (amount == 0) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L69)

    81: if (amount == 0) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L81)

 ##

### [G-11] SETTING THE CONSTRUCTOR TO PAYABLE

Saves ~13 gas per instance

FILE : 2023-03-wenwin/src/Lottery.sol

  84 :  constructor(

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L84)

FILE : 2023-03-wenwin/src/LotterySetup.sol

   41:  constructor(LotterySetupParams memory lotterySetupParams) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L41)

FILE : 2023-03-wenwin/src/staking/Staking.sol

  22:  constructor(

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L22)

FILE : 2023-03-wenwin/src/staking/StakedTokenLock.sol

   16: constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L16)

FILE : 2023-03-wenwin/src/VRFv2RNSource.sol

   13:   constructor(

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/VRFv2RNSource.sol#L13)

##

### [G-12] Use assembly to write address storage values

FILE : 2023-03-wenwin/src/staking/Staking.sol

      41: lottery = _lottery;
      42: rewardsToken = _rewardsToken;
      43: stakingToken = _stakingToken;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L42-L43)

FILE  : 2023-03-wenwin/src/staking/StakedTokenLock.sol

   18: stakedToken = IStaking(_stakedToken);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L18)

FILE : 2023-03-wenwin/src/LotterySetup.sol

##

### [G-13] INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

FILE : 2023-03-wenwin/src/LotterySetup.sol

   160: function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L160)

FILE : 2023-03-wenwin/src/ReferralSystem.sol

   156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L156)

   161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

##

### [G-14] Use constants instead of type(uintx).max

type(uint16).max . it uses more gas in the distribution process and also for each transaction than constant usage.

FILE : 2023-03-wenwin/src/LotterySetup.sol

    126: uint256 mask = uint256(type(uint16).max) << (winTier * 16);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L126)

##

### [G-15] Internal functions not called by contract can be removed to save the gas

FILE : 2023-03-wenwin/src/ReferralSystem.sol

   function referralRegisterTickets(
        uint128 currentDraw,
        address referrer,
        address player,
        uint256 numberOfTickets
    )
        internal

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L52-L58)

    87: function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L87)

##

### [G-16] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

FILE :2023-03-wenwin/src/staking/StakedTokenLock.sol

   47:  stakedToken.transfer(msg.sender, amount);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L47)

   55:  rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L55)

   34: stakedToken.transferFrom(msg.sender, address(this), amount);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L34)

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