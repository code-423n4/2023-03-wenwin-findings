##

### [1] For modern and more readable code; update import usages

Description

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

Recommendation

import {contract1 , contract2} from "filename.sol";

FILE : 2023-03-wenwin/src/LotteryToken.sol

   5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
   6: import "src/interfaces/ILotteryToken.sol";
   7: import "src/LotteryMath.sol";

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryToken.sol#L5-L7)

FILE: 2023-03-wenwin/src/VRFv2RNSource.sol

   5: import "@chainlink/contracts/src/v0.8/VRFV2WrapperConsumerBase.sol";
   6: import "src/interfaces/IVRFv2RNSource.sol";
   7: import "src/RNSourceBase.sol";

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/VRFv2RNSource.sol#L5-L7)

FILE : 2023-03-wenwin/src/staking/Staking.sol

   import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
   import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
   import "src/interfaces/ILottery.sol";
   import "src/LotteryMath.sol";
   import "src/staking/interfaces/IStaking.sol";

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L5-L9)

FILE : 2023-03-wenwin/src/LotterySetup.sol

   import "@openzeppelin/contracts/utils/math/Math.sol";
   import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
   import "src/PercentageMath.sol";
   import "src/LotteryToken.sol";
   import "src/interfaces/ILotterySetup.sol";
   import "src/Ticket.sol";

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L5-L10)

FILE : 2023-03-wenwin/src/Lottery.sol

    import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
    import "@openzeppelin/contracts/utils/math/Math.sol";
    import "src/ReferralSystem.sol";
    import "src/RNSourceController.sol";
    import "src/staking/Staking.sol";
    import "src/LotterySetup.sol";
    import "src/TicketUtils.sol";

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L5-L11)

FILE : 2023-03-wenwin/src/Ticket.sol

    import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
    import "src/interfaces/ITicket.sol";

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L5-L6)

FILE : 2023-03-wenwin/src/RNSourceController.sol

    import "@openzeppelin/contracts/access/Ownable2Step.sol";
    import "src/interfaces/IRNSource.sol";
    import "src/interfaces/IRNSourceController.sol";

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L5-L7)


##

### [2] Use safeMint Instead plain mint functions 

Some benefits of using safeMint instead of mint

 Prevents integer overflow, Protects against reentrancy attacks, Improved security,Better code quality

FILE : 2023-03-wenwin/src/LotteryToken.sol

   19: _mint(msg.sender, INITIAL_SUPPLY);

   26:  _mint(account, amount);

### [3] MIXING AND OUTDATED COMPILER

The pragma version used are:

pragma solidity ^0.8.7;

Note that mixing pragma is not recommended. Because different compiler versions have different meanings and behaviors, it also significantly raises maintenance costs. As a result, depending on the compiler version selected for any given file, deployed contracts may have security issues.

The minimum required version must be 0.8.19; otherwise, contracts will be affected by the following important bug fixes:

0.8.14:

ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.
0.8.15

Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.
0.8.16

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.
0.8.17

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.
Apart from these, there are several minor bug fixes and improvements

FILE : 2023-03-wenwin/src/LotteryToken.sol

    3: pragma solidity 0.8.19;

FILE: 2023-03-wenwin/src/VRFv2RNSource.sol
 
    3 : pragma solidity ^0.8.7;

FILE : 2023-03-wenwin/src/staking/Staking.sol

  3: pragma solidity 0.8.19;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L3)

##

### [4] FOR FUNCTIONS, FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

Description
The bellow codes don’t follow Solidity’s standard naming convention,

FILE: 2023-03-wenwin/src/VRFv2RNSource.sol

internal functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

  28: function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {
  32: function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/VRFv2RNSource.sol#L28-L32)

FILE : 2023-03-wenwin/src/Lottery.sol

  203 : function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {

  279: function requireFinishedDraw(uint128 drawId) internal view override {
  
  285: function mintNativeTokens(address mintTo, uint256 amount) internal override {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L203)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L23)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L19)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceBase.sol#L33)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L52)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L76)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L87)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/PercentageMath.sol#L17-L19)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/PercentageMath.sol#L22-L24)

FILE : 2023-03-wenwin/src/Lottery.sol

 private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

   function registerTicket(
        uint128 drawId,
        uint120 ticket,
        address frontend,
        address referrer
    )
        private

    271 : function returnUnclaimedJackpotToThePot() private {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L181-L187)

  

##

### [5] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

FILE: 2023-03-wenwin/src/VRFv2RNSource.sol

  28: function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {
  32: function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/VRFv2RNSource.sol#L28-L32)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L120)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L152)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L156)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L164)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L110-L119)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L144)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L151)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L159)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L170)


##

### [6]  MISSING CHECKS FOR ADDRESS(0X0) WHEN ASSIGNING VALUES TO ADDRESS STATE VARIABLES

FILE : 2023-03-wenwin/src/staking/StakedTokenLock.sol

     18: stakedToken = IStaking(_stakedToken);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L18)

FILE : 2023-03-wenwin/src/RNSourceBase.sol

    constructor(address _authorizedConsumer) {
        authorizedConsumer = _authorizedConsumer;
    }
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceBase.sol#L11-L13)

##

### [7] LACK OF CHECKS THE INTEGER RANGES

The following methods lack of input validations for _depositDeadline and _lockDuration before assigning to state variables. 

FILE : 2023-03-wenwin/src/staking/StakedTokenLock.sol

        constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
        _transferOwnership(msg.sender);
        stakedToken = IStaking(_stakedToken);
        rewardsToken = stakedToken.rewardsToken();
        depositDeadline = _depositDeadline;
        lockDuration = _lockDuration;
        }

   (https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L20-L21)

Amount not checked before proceed to transfer 

       24: function deposit(uint256 amount) external override onlyOwner {

      37:  function withdraw(uint256 amount) external override onlyOwner {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L37)

FILE : 2023-03-wenwin/src/Lottery.sol

        constructor(
        LotterySetupParams memory lotterySetupParams,
        uint256 playerRewardFirstDraw,
        uint256 playerRewardDecreasePerDraw,
        uint256[] memory rewardsToReferrersPerDraw,
        uint256 maxRNFailedAttempts,
        uint256 maxRNRequestDelay
        )

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L84-L108)

FILE : 2023-03-wenwin/src/ReferralSystem.sol

      constructor(
        uint256 _playerRewardFirstDraw,
        uint256 _playerRewardDecreasePerDraw,
        uint256[] memory _rewardsToReferrersPerDraw
        ) {
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L27-L31)

Recommended mitigations:

 perform possible input validations before assigning to state variables Like (> 0, > low range, < upper range)

##

### [8] LOSS OF PRECISION DUE TO ROUNDING

FILE : 2023-03-wenwin/src/staking/Staking.sol

     62 : return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + 
     rewards[account];

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L62)

    58: return rewardPerTokenStored + (unclaimedRewards * 1e18 / _totalSupply);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L58)

FILE : 2023-03-wenwin/src/ReferralSystem.sol

          99: referrerRewardPerDrawForOneTicket[drawFinalized] =
              referrerRewardForDraw / totalTicketsForReferrersPerCurrentDraw;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L99-L100)
       
      105:  playerRewardsPerDrawForOneTicket[drawFinalized] = playerRewardForDraw / ticketsSoldDuringDraw;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L105) 

    123:   return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);

    127 :  return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);

FILE : 2023-03-wenwin/src/LotteryMath.sol

    84: bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L84)

##

### [9]  CONSTANTS SHOULD BE DEFINED RATHER THAN USING MAGIC NUMBERS

  It is bad practice to use numbers directly in code without explanation

FILE : 2023-03-wenwin/src/LotterySetup.sol

    51: if (lotterySetupParams.selectionMax >= 120) {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L51)
 
      if (
            lotterySetupParams.expectedPayout < lotterySetupParams.ticketPrice / 100
                || lotterySetupParams.expectedPayout >= lotterySetupParams.ticketPrice
        )

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L54-L57)

       if (
            lotterySetupParams.selectionSize > 16 || lotterySetupParams.selectionSize >= lotterySetupParams.selectionMax
        )

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L60-L62)
   
        79: uint256 tokenUnit = 10 ** IERC20Metadata(address(lotterySetupParams.token)).decimals();
        80: minInitialPot = 4 * tokenUnit;
        81: jackpotBound = 2_000_000 * tokenUnit;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L79-L81)

     126: uint256 mask = uint256(type(uint16).max) << (winTier * 16);
     127: uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L126-L127)

FILE : 2023-03-wenwin/src/ReferralSystem.sol

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L117)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L121)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L123)   
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L125)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L127)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L129)

##

### [10] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
(https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

Recommendation
NatSpec comments should be increased in Contracts

##

### [11] NATSPEC IS MISSING

Description
NatSpec is missing for the following functions ,constructor and modifier:

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L152)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L156)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L132)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L120)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L110-L119)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L110-L120)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L133)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L144)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L104-L118)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L151)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L120)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L159)

##

### [12] Shorter the inheritance list 

FILE : 2023-03-wenwin/src/Lottery.sol

   21: contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceController {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L21)

##

### [13] INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS

(https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

If Return parameters are declared, you must prefix them with ”/// @return”.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

Recommended Mitigation Steps
Include return parameters in NatSpec comments

FILE : 2023-03-wenwin/src/Lottery.sol

  /// @dev Registers the ticket in the system. To be called when user is buying the ticket.
    /// @param drawId Draw identifier ticket is bought for.
    /// @param ticket Combination packed as uint120.
    function registerTicket(
        uint128 drawId,
        uint120 ticket,
        address frontend,
        address referrer
    )
        private
        beforeTicketRegistrationDeadline(drawId)
        requireValidTicket(ticket)
        returns (uint256 ticketId)

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L178-L190)

##
###[14] USE REQUIRE INSTEAD OF ASSERT

Description
Assert should not be used except for tests, require should be used.

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

assert() and require();
The big difference between the two is that the assert()function when false, uses up all the remaining gas and reverts all the changes made.
Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay.
This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256).Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. There’s a mistake”.

FILE : 2023-03-wenwin/src/TicketUtils.sol

     99: assert((winTier <= selectionSize) && (intersection == uint256(0)));

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L99)

FILE : 2023-03-wenwin/src/LotterySetup.sol

    147: assert(initialPot > 0);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L147)

##

### [16] NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES

FILE : 2023-03-wenwin/src/staking/StakedTokenLock.sol

  3: pragma solidity ^0.8.17;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L3)

##

### [17] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256,Uint8,Uint128,uint120

FILE : 2023-03-wenwin/src/TicketUtils.sol

     5:  uint256 currentSelectionCount = uint256(selectionMax);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L55)
   
    58:   numbers[i] = uint8(randomNumber % currentSelectionCount);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L58)

    74: ticket |= ((uint120(1) << currentNumber));

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L74)

   96: winTier += uint8(intersection & uint120(1));

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L96)

FILE : 2023-03-wenwin/src/LotteryMath.sol

    48:   newProfit = oldProfit + int256(ticketsSalesToPot);

    64 :  excessPotInt -= int256(fixedJackpotSize);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L64)

##

### [18] A single point of failure

Impact

The onlyOwner role has a single point of failure and onlyOwner can use critical a few functions.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

onlyOwner functions:

FILE : 2023-03-wenwin/src/staking/StakedTokenLock.sol

  24: function deposit(uint256 amount) external override onlyOwner {

  37: function withdraw(uint256 amount) external override onlyOwner {

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L37)

##

### [19] Missing Event for critical parameters init and change 

FILE : 2023-03-wenwin/src/Lottery.sol

constructor(
        LotterySetupParams memory lotterySetupParams,
        uint256 playerRewardFirstDraw,
        uint256 playerRewardDecreasePerDraw,
        uint256[] memory rewardsToReferrersPerDraw,
        uint256 maxRNFailedAttempts,
        uint256 maxRNRequestDelay
    )
        Ticket()
        LotterySetup(lotterySetupParams)
        ReferralSystem(playerRewardFirstDraw, playerRewardDecreasePerDraw, rewardsToReferrersPerDraw)
        RNSourceController(maxRNFailedAttempts, maxRNRequestDelay)
    {
 When new Ticket, LotterySetup initialized no events called

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L84-L108)

##

### [20] Function writing that does not comply with the Solidity Style Guide

Context
All Contracts

Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

(https://docs.soliditylang.org/en/v0.8.17/style-guide.html)

Functions should be grouped according to their visibility and ordered:

  constructor
  receive function (if exists)
  fallback function (if exists)
  external
  public
  internal
  private
  within a grouping, place the view and pure functions last

##

### [21] Need Fuzzing test

FILE :2023-03-wenwin/src/TicketUtils.sol

  26:  unchecked {

  93: unchecked {

Description
In total 1 contract, 2 unchecked are used, the functions used are critical. For this reason, there must be fuzzing tests in the tests of the project. Not seen in tests.

Recommendation

Use should fuzzing test like Echidna.

##

### [22] Tokens accidentally sent to the contract cannot be recovered

It can’t be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

Recommended Mitigation Steps :

Add this code:

 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

##

### [23] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

FILE : 2023-03-wenwin/src/PercentageMath.sol

   uint256 public constant PERCENTAGE_BASE = 100_000;

    /// @dev percentage number representing 1%.
    uint256 public constant ONE_PERCENT = 1000;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/PercentageMath.sol#L8-L11)

FILE : 2023-03-wenwin/src/LotteryMath.sol


   /// @dev percentage of ticket price being paid for staking reward
    uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;
    /// @dev percentage of ticket price being paid to frontend operator
    uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;
    /// @dev Percentage of the ticket price that goes to the pot
    uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;
    /// @dev safety margin used to calculate excess pot, in percentage
    uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;
    /// @dev Percentage of excess pot reserved for bonus
    uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;
    /// @dev Number of lottery draws per year
    uint128 public constant DRAWS_PER_YEAR = 52;

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L13-L24)

FILE : 2023-03-wenwin/src/LotterySetup.sol

  36:  uint256 private constant BASE_JACKPOT_PERCENTAGE = 30_030; // 30.03%

##

### [24] Use .call instead of .transfer to send ether

The reason why call() is preferred over transfer() in some cases is that call() provides more control and flexibility, and allows for error handling and recovery in case of unexpected behavior

FILE :2023-03-wenwin/src/staking/StakedTokenLock.sol

   47:  stakedToken.transfer(msg.sender, amount);

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L47)

   55:  rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L55)

##




    


   
  








NC-1	Typos	63
L-1	Empty Function Body - Consider commenting why	1
L-2	Unsafe ERC20 operation(s)	3
