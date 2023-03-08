##

### [1] USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’

Using named imports in this way can make your code easier to read and understand, as well as reduce the risk of naming conflicts and make it easier to maintain and modify your code in the future. Additionally, importing only the specific functions or variables that you need can help reduce gas costs and improve security by reducing the amount of unnecessary code that is deployed on the blockchain

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

##

### [9] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
(https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

Recommendation
NatSpec comments should be increased in Contracts

##

### [11] NATSPEC IS MISSING

Description
NatSpec is missing for the following functions , constructor and modifier:

(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L152)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L156)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L132)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L120)
(https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L110-L119)
()
()
()
()
()
()
()

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

    


   
  








NC-1	Typos	63
L-1	Empty Function Body - Consider commenting why	1
L-2	Unsafe ERC20 operation(s)	3
