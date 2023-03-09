### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] |Use hardcode address instead ``address(this)`` | 5 |
| [G-02] |Gas can be saved by using functions instead of using a modifier| 6 |
| [G-03] |Structs can be packed into fewer storage slots |1 |
| [G-04] |Pack state variables | 1 |
| [G-05] | Remove unused code |1|
| [G-06] |Use ``calldata`` instead of ``memory`` for function arguments that do not get mutated | 7 |
| [G-07] |Using ``delete``  instead of setting mapping/state variable ``0`` saves gas | 8 |
| [G-08] | Using ``delete`` to set ``bool`` values to their default values saves gas | 2 |
| [G-09] | Multiple ``address``/ID mappings can be combined into a single ``mapping`` of an ``address``/ID to a ``struct``, where appropriate  | 5 |
| [G-10] |Multiple accesses of a mapping/array should use a local variable cache | 10 |
| [G-11] | Avoid using ``state variable`` in emit |2 |
| [G-12] |Change ``public`` function visibility to ``external`` | 2 |
| [G-13] |``internal`` functions only called once can be inlined to save gas  | 11 |
| [G-14] |Do not calculate constants variables |5 |
| [G-15] |Use ``assembly`` to write _address storage values_  | 2 |
| [G-16] |Setting the _constructor_ to `payable` |10 |
| [G-17] |_Functions guaranteed to revert_ when called by normal users can be marked ``payable`` | 4 |
| [G-18] |Missing `zero-address` check in `constructor` |3 |
| [G-19] |Use assembly to check for ``address(0)``|10|
| [G-20] |Add ``unchecked {}`` for subtractions where the operands cannot underflow because of a previous ``require`` or ``if`` statement | 1 |
| [G-21] |`x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables | 3 |
| [G-22] |Use ``require`` instead of ``assert`` | 2 |
| [G-23] |Use nested if and, avoid multiple check combinations | 2 |
| [G-24] |Sort Solidity operations using short-circuit mode| 6 |
| [G-25] |Optimize names to save gas  | All contracts |
| [G-26] |Use ``< (>)`` instead of ``<= ( >=)`` | 2 |
| [G-27] |``>=`` costs less gas than ``>``| 2 |
| [G-28] |Upgrade Solidity's optimizer |  |

Total 28 issues

### [G-01] Use hardcode address instead ``address(this)``

Instead of ``address(this)``, it is more gas-efficient to pre-calculate and use the address with a hardcode. Foundry's ``script.sol`` and solmate````LibRlp.sol`` contracts can do this.

Reference:
https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824


5 results - 4 files:
```solidity
src\Lottery.sol:

  130:         rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L130


```solidity
src\LotterySetup.sol:

  140:         uint256 raised = rewardToken.balanceOf(address(this));

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L140


```solidity
src\staking\StakedTokenLock.sol:

  34:         stakedToken.transferFrom(msg.sender, address(this), amount);

  55:         rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L34


```solidity
src\staking\Staking.sol:
  
  73:         stakingToken.safeTransferFrom(msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L34

### [G-02] Gas can be saved by using functions instead of using a modifier

When the in-scope and out-of-scope contracts are examined; It has been observed that the modifiers in the **Lottery.sol (4 pieces)** and **LotterySetup.sol (2 pieces)** contracts are used only once in the ``Lottery.sol```contract.

6 results - 2 files:
```solidity
src\Lottery.sol:

  44:     modifier requireValidTicket(uint256 ticket) {
  45:         if (!ticket.isValidTicket(selectionSize, selectionMax)) {
  46:             revert InvalidTicket();
  47:         }
  48:         _;
  49:     }
 

  52:     modifier whenNotExecutingDraw() {
  53:         if (drawExecutionInProgress) {
  54:             revert DrawAlreadyInProgress();
  55:         }
  56:         _;
  57:     }


  60:     modifier onlyWhenExecutingDraw() {
  61:         if (!drawExecutionInProgress) {
  62:             revert DrawNotInProgress();
  63:         }
  64:         _;
  65:     }

  69:     modifier onlyTicketOwner(uint256 ticketId) {
  70:         if (ownerOf(ticketId) != msg.sender) {
  71:             revert UnauthorizedClaim(ticketId, msg.sender);
  72:         }
  73:         _;
  74:     }

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L44-L74

```solidity
src\LotterySetup.sol:

  104:     modifier requireJackpotInitialized() {
  105:         // slither-disable-next-line incorrect-equality
  106:         if (initialPot == 0) {
  107:             revert JackpotNotInitialized();
  108:         }
  109:         _;
  110:     }

  112:     modifier beforeTicketRegistrationDeadline(uint128 drawId) {
  113:         // slither-disable-next-line timestamp
  114:         if (block.timestamp > ticketRegistrationDeadline(drawId)) {
  115:             revert TicketRegistrationClosed(drawId);
  116:         }
  117:         _;
  118:     }

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L104-L118


My suggestion to the project team is to remove these modifiers and make these checks with functions. This will provide significant gas savings.


```diff

src\LotterySetup.sol:

- 104:     modifier requireJackpotInitialized() {
- 105:         // slither-disable-next-line incorrect-equality
- 106:         if (initialPot == 0) {
- 107:             revert JackpotNotInitialized();
- 108:         }
- 109:         _;
- 110:     }

+ function requireJackpotInitialized() internal {
+        if (initialPot == 0) {
+            revert JackpotNotInitialized();
+        }
+    }

```

### [G-03] Structs can be packed into fewer storage slots

The " LotterySetupParams " structure can be packaged as suggested below.

**1 slot win 1 * 2k  =  2k gas saved**

```solidity
src\interfaces\ILotterySetup.sol:

  62  /// @dev Parameters used to setup a new lottery
  63: struct LotterySetupParams {
  64:     /// @dev Token to be used as reward token for the lottery
  65:     IERC20 token;                                                         
  66:     /// @dev Parameters of the draw schedule for the lottery
  67:     LotteryDrawSchedule drawSchedule;
  68:     /// @dev Price to pay for playing single game (including fee)
  69:     uint256 ticketPrice;
  70:     /// @dev Count of numbers user picks for the ticket
  71:     uint8 selectionSize;
  72:     /// @dev Max number user can pick
  73:     uint8 selectionMax;
  74:     /// @dev Expected payout for one ticket, expressed in `rewardToken`
  75:     uint256 expectedPayout;
  76:     /// @dev Array of fixed rewards per each non jackpot win
  77:     uint256[] fixedRewards;
  78: }

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L62-L78

```diff
src\interfaces\ILotterySetup.sol:

  62  /// @dev Parameters used to setup a new lottery
  63: struct LotterySetupParams {
- 64:     /// @dev Token to be used as reward token for the lottery
- 65:     IERC20 token;
  66:     /// @dev Parameters of the draw schedule for the lottery
  67:     LotteryDrawSchedule drawSchedule;
  68:     /// @dev Price to pay for playing single game (including fee)
  69:     uint256 ticketPrice;
+ 64:     /// @dev Token to be used as reward token for the lottery
+ 65:     IERC20 token;
  70:     /// @dev Count of numbers user picks for the ticket
  71:     uint8 selectionSize;
  72:     /// @dev Max number user can pick
  73:     uint8 selectionMax;
  74:     /// @dev Expected payout for one ticket, expressed in `rewardToken`
  75:     uint256 expectedPayout;
  76:     /// @dev Array of fixed rewards per each non jackpot win
  77:     uint256[] fixedRewards;
  78: }

```

### [G-04] Pack state variables

State variable packaging in ** RNSourceController.sol ** contract  (**1 * 2k = 2k gas saved**)

Since the ``uint64`` species will be sufficient for the next 532 years, the ``uint256 Lastrequesttimestmp`` value can be changed to ``uint64 lastrequesttimestamp``. In this way, 1 slot savings will be provided.

```solidity
src\RNSourceController.sol:

  15:     uint256 public override lastRequestTimestamp;

  16:     bool public override lastRequestFulfilled = true;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L15_l16

```diff
src\RNSourceController.sol:

- 15:     uint256 public override lastRequestTimestamp;
+ 15:     uint64 public override lastRequestTimestamp;

  16:     bool public override lastRequestFulfilled = true;

```

### [G-05] Remove unused code

The ``ReferralRequirementFactorType`` enum and ``MinimumReferralsRequirement`` struct in the `IReferralSystemDynamic.sol` contract are not used in any in-scope or out-of-scope contract. 

I recommend removing this interface (**IReferralSystemDynamic.sol**).

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystemDynamic.sol


### [G-06] Use ``calldata`` instead of ``memory`` for function arguments that do not get mutated

Mark data types as ``calldata`` instead of ``memory`` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as ``calldata``. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies ``memory`` storage.


7 results - 4 files:
```solidity
src\Lottery.sol:

  85:    LotterySetupParams memory lotterySetupParams,

  88:    uint256[] memory rewardsToReferrersPerDraw,

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L85


```solidity
src\LotterySetup.sol:

  41:    constructor(LotterySetupParams memory lotterySetupParams) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L41


```solidity
src\ReferralSystem.sol:

  30:    uint256[] memory _rewardsToReferrersPerDraw

  76:    function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L30


```solidity
src\staking\Staking.sol:

  26:     string memory name,
  
  27:     string memory symbol

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L26-L27


### [G-07] Using ``delete``  instead of setting mapping/state variable ``0`` saves gas

8 results - 4 files:
```solidity
src\Lottery.sol:

  255:    frontendDueTicketSales[beneficiary] = 0;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L255

```solidity
src\ReferralSystem.sol:

  142:    unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;

  148:    unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L142

```solidity
src\RNSourceController.sol:

   52:    failedSequentialAttempts = 0;

   53:    maxFailedAttemptsReachedAt = 0;

   99:    failedSequentialAttempts = 0;

  100:    maxFailedAttemptsReachedAt = 0;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L52-L53

```solidity
src\staking\Staking.sol:

  95:     rewards[msg.sender] = 0;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L95

**Recommendation code:**
```diff
src\staking\Staking.sol:

- 95:     rewards[msg.sender] = 0;
+ 95:     delete rewards[msg.sender];

```

### [G-08] Using ``delete`` to set ``bool`` values to their default values saves gas

2 results - 2 files:
```solidity
src\Lottery.sol:

  225:         drawExecutionInProgress = false;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L225


```solidity

src\RNSourceController.sol:

  108:         lastRequestFulfilled = false;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L108

### [G-09] Multiple ``address``/ID mappings can be combined into a single ``mapping`` of an ``address``/ID to a ``struct``, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.


5 results 2 files:
```solidity
src\Lottery.sol:

  27:     mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
  36:     mapping(uint128 => uint120) public override winningTicket;
  37:     mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L27

```solidity
src\ReferralSystem.sol:

19:     mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;
25:     mapping(uint128 => uint256) public override minimumEligibleReferrals;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L19

```solidity
src\staking\Staking.sol:

  19:     mapping(address => uint256) public override userRewardPerTokenPaid;
  20:     mapping(address => uint256) public override rewards;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L19-L20


### [G-10] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata.

10 results - 3 files:
```solidity
src\LotterySetup.sol:

  // @audit rewards[winTier]
  170:             uint16 reward = uint16(rewards[winTier] / divisor);
  171:             if ((rewards[winTier] % divisor) != 0) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L170-L171

```solidity
src\ReferralSystem.sol:

  // @audit unclaimedTickets[currentDraw][referrer]
  62:             if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
  63:                 if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
  65:                         unclaimedTickets[currentDraw][referrer].referrerTicketCount;
  69:             unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L62-L63

```solidity
src\RNSourceBase.sol:

  // @audit requests[requestId]
  34:         if (requests[requestId].status == RequestStatus.None) {
  37:         if (requests[requestId].status == RequestStatus.Fulfilled) {
  41:         requests[requestId].randomNumber = randomNumber;
  42:         requests[requestId].status = RequestStatus.Fulfilled;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L34


### [G-11] Avoid using ``state variable`` in emit

Using the current memory value here will be more gas-optimized.


```solidity
src\Lottery.sol:
  133:     function executeDraw() external override whenNotExecutingDraw {
  134:         // slither-disable-next-line timestamp
  135:         if (block.timestamp < drawScheduledAt(currentDraw)) {
  136:             revert ExecutingDrawTooEarly();
  137:         }
  138:         returnUnclaimedJackpotToThePot();
  139:         drawExecutionInProgress = true;
  140:         requestRandomNumber();
  141:         emit StartedExecutingDraw(currentDraw);
  142:     }

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L133-L142

```diff
src\Lottery.sol:

  133:     function executeDraw() external override whenNotExecutingDraw {
  134:         // slither-disable-next-line timestamp
+              uint128 _currentDraw = currentDraw;
- 135:         if (block.timestamp < drawScheduledAt(currentDraw)) {
+ 135:         if (block.timestamp < drawScheduledAt(_currentDraw)) {
  136:             revert ExecutingDrawTooEarly();
  137:         }
  138:         returnUnclaimedJackpotToThePot();
  139:         drawExecutionInProgress = true;
  140:         requestRandomNumber();
- 141:         emit StartedExecutingDraw(currentDraw);
+ 141:         emit StartedExecutingDraw(_currentDraw);
  142:     }

```



```solidity
src\RNSourceController.sol:

  106:     function requestRandomNumberFromSource() private {
  107:         lastRequestTimestamp = block.timestamp;
  108:         lastRequestFulfilled = false;
  109: 
  110:         // slither-disable-start uninitialized-local
  111:         // See Slither issue: https://github.com/crytic/slither/issues/511
  112:         try source.requestRandomNumber() {
  113:             emit SuccessfulRNRequest(source);
  114:         } catch Error(string memory reason) {
  115:             emit FailedRNRequest(source, bytes(reason));
  116:         } catch (bytes memory reason) {
  117:             emit FailedRNRequest(source, reason);
  118:         }
  119:         // slither-disable-end uninitialized-local
  120:     }
  121: }

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L106-L121

```diff
src\RNSourceController.sol:

  106:     function requestRandomNumberFromSource() private {
  107:         lastRequestTimestamp = block.timestamp;
  108:         lastRequestFulfilled = false;
  109: 
  110:         // slither-disable-start uninitialized-local
  111:         // See Slither issue: https://github.com/crytic/slither/issues/511
+              address _source = source;
- 112:         try source.requestRandomNumber() {
+ 112:         try _source.requestRandomNumber() {  
- 113:             emit SuccessfulRNRequest(source);
+ 113:             emit SuccessfulRNRequest(_source);
  114:         } catch Error(string memory reason) {
- 115:             emit FailedRNRequest(source, bytes(reason));
+ 115:             emit FailedRNRequest(_source, bytes(reason));
  116:         } catch (bytes memory reason) {
- 117:             emit FailedRNRequest(source, reason);
+ 117:             emit FailedRNRequest(_source, reason);
  118:         }
  119:         // slither-disable-end uninitialized-local
  120:     }
  121: }

```

### [G-12] Change ``public`` function visibility to ``external``

Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

2 results - 2 files:
```solidity

src\Lottery.sol:

  234:    function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L234


```solidity
src\LotterySetup.sol:

  120:    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120


### [G-13] ``internal`` functions only called once can be inlined to save gas

Not inlining costs **20** to **40** gas because of two extra ``JUMP`` instructions and additional stack operations needed for function calls.

11 results - 7 files:
```solidity
src\Lottery.sol:

  203:     function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {
  
  279:     function requireFinishedDraw(uint128 drawId) internal view override {
  
  285:     function mintNativeTokens(address mintTo, uint256 amount) internal override {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L203

```solidity
src\PercentageMath.sol:

  17:     function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
  
  22:     function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L17

```solidity
src\ReferralSystem.sol:

  87:     function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L87

```solidity
src\RNSourceBase.sol:

  33:     function fulfill(uint256 requestId, uint256 randomNumber) internal {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L33

```solidity
src\Ticket.sol:

  19:     function markAsClaimed(uint256 ticketId) internal {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L19


```solidity
src\VRFv2RNSource.sol:

  28:     function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {
  
  32:     function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L28

```solidity
src\staking\Staking.sol:

  108:     function _beforeTokenTransfer(address from, address to, uint256) internal override {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L108


### [G-14] Do not calculate constants variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

5 results - 1 file:
```solidity
src\LotteryMath.sol:

  14:     uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;

  16:     uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;

  18:     uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;
  
  20:     uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;
 
  22:     uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L14-L22


```diff
src\LotteryMath.sol:

- 14:     uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;
  // @audit uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT
+ 14:     uint256 public constant STAKING_REWARD = 20_000;


- 16:     uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;
  // @audit uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT
+ 16:     uint256 public constant FRONTEND_REWARD = 10_000;


- 18:     uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;
  // @audit TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD
+ 18:     uint256 public constant TICKET_PRICE_TO_POT = 70_000;
  
- 20:     uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;
  // @audit SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT
+ 20:     uint256 public constant SAFETY_MARGIN = 67_000;
 

- 22:     uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;
  // @audit EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT
+ 22:     uint256 public constant EXCESS_BONUS_ALLOCATION = 50_000;
  
```

### [G-15] Use ``assembly`` to write _address storage values_ 

2 results - 1 file:
```solidity
src\RNSourceController.sol:

  77:     function initSource(IRNSource rnSource) external override onlyOwner {
  85:         source = rnSource;

 
  89:     function swapSource(IRNSource newSource) external override onlyOwner {
  98:         source = newSource;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L85
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L98


**Recommendation Code:**
```diff
src\RNSourceController.sol:

  77:     function initSource(IRNSource rnSource) external override onlyOwner {
- 85:         source = rnSource;
+                  assembly {                      
+                      sstore(source .slot, rnSource)
+                  }                               
}                               
          
```

### [G-16] Setting the _constructor_ to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.

10 results - 10 files:
```solidity
src\Lottery.sol:

  84:     constructor(

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L84

```solidity
src\LotterySetup.sol:

  41:     constructor(LotterySetupParams memory lotterySetupParams) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L41

```solidity
src\LotteryToken.sol:

  17:     constructor() ERC20("Wenwin Lottery", "LOT") {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L17

```solidity
src\ReferralSystem.sol:
  
  27:     constructor(

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L27

```solidity
src\RNSourceBase.sol:
 
  11:     constructor(address _authorizedConsumer) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L11

```solidity
src\RNSourceController.sol:

  26:     constructor(uint256 _maxFailedAttempts, uint256 _maxRequestDelay) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L26

```solidity
src\Ticket.sol:

  17:     constructor() ERC721("Wenwin Lottery Ticket", "WLT") { }
 
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L17

```solidity
src\VRFv2RNSource.sol:
 
  13:     constructor(

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L13

```solidity
src\staking\StakedTokenLock.sol:
  
  16:     constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L16

```solidity
src\staking\Staking.sol:
 
  22:     constructor(

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L22


**Recommendation:**
Set the constructor to ```payable```

### [G-17] _Functions guaranteed to revert_ when called by normal users can be marked ``payable``

If a function modifier such as ``onlyOwner`` is used, the function will revert if a normal user tries to pay the function. Marking the function as ``payable`` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

4 results - 2 files:
```solidity

src\RNSourceController.sol:
    
  77:     function initSource(IRNSource rnSource) external override onlyOwner {

  89:     function swapSource(IRNSource newSource) external override onlyOwner {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L77


```solidity
src\staking\StakedTokenLock.sol:
    
  24:     function deposit(uint256 amount) external override onlyOwner {
 
  37:     function withdraw(uint256 amount) external override onlyOwner {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L24


###  [G-18] Missing `zero-address` check in `constructor`

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

3 results - 3 files:
```solidity
src\RNSourceBase.sol:

  11:     constructor(address _authorizedConsumer) {
  12:         authorizedConsumer = _authorizedConsumer;
  13:     }

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L11-L13

```solidity
src\VRFv2RNSource.sol:
 
  13:     constructor(
  14:         address _authorizedConsumer,
  15:         address _linkAddress,
  16:         address _wrapperAddress,
  17:         uint16 _requestConfirmations,
  18:         uint32 _callbackGasLimit
  19:     )
  20:         RNSourceBase(_authorizedConsumer)
  21:         VRFV2WrapperConsumerBase(_linkAddress, _wrapperAddress)
  22:     {
  23:         requestConfirmations = _requestConfirmations;
  24:         callbackGasLimit = _callbackGasLimit;
  25:     }

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L13-L25


```solidity
src\staking\StakedTokenLock.sol:
  
  16:     constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
  17:         _transferOwnership(msg.sender);
  18:         stakedToken = IStaking(_stakedToken);
  19:         rewardsToken = stakedToken.rewardsToken();
  20:         depositDeadline = _depositDeadline;
  21:         lockDuration = _lockDuration;
  22:     }

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L16-L22


### [G-19] Use assembly to check for ``address(0)``
10 results - 4 files:
```solidity
src\LotterySetup.sol:

  42:    if (address(lotterySetupParams.token) == address(0)) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L42

```solidity
src\RNSourceController.sol:

  78:    if (address(rnSource) == address(0)) {

  81:    if (address(source) != address(0)) {

  90:    if (address(newSource) == address(0)) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L78

```solidity
src\staking\Staking.sol:

  31:    if (address(_lottery) == address(0)) {

  34:    if (address(_rewardsToken) == address(0)) {
 
  37:    if (address(_stakingToken) == address(0)) {

  109:   if (from != address(0)) {

  113:   if (to != address(0)) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L31

```solidity
src\ReferralSystem.sol:

  60:    if (referrer != address(0)) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L60

### [G-20] Add ``unchecked {}`` for subtractions where the operands cannot underflow because of a previous ``require`` or ``if`` statement

```
require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a } 
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
```
This will stop the check for overflow and underflow so it will save gas.


1 result - 1 file:
```solidity
src\Lottery.sol:

  272:         if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {
  273:             uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L273

### [G-21] `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables

3 results - 2 files:
```solidity
src\Lottery.sol:

  275:    currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275


```solidity
src\staking\StakedTokenLock.sol:

  30:     depositedBalance += amount;

  43:     depositedBalance -= amount;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30


`x += y (x -= y)` costs more gas than `x = x  + y (x = x - y)` for state variables.


### [G-22] Use ``require`` instead of ``assert``

The assert() and require() functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made.

2 results - 2 files:
```solidity
src\LotterySetup.sol:

  147:         assert(initialPot > 0);

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147


```solidity
src\TicketUtils.sol:

  99:             assert((winTier <= selectionSize) && (intersection == uint256(0)));

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99

### [G-23] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

2 results - 2 files:
```solidity
src\LotteryMath.sol:

  83:         if (excessPot > 0 && ticketsSold > 0) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L83


```solidity
src\staking\StakedTokenLock.sol:

  39:         if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L39



**Recomendation Code:**
```diff
src\staking\StakedTokenLock.sol:

- 39:         if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {
+ 39:         if (block.timestamp > depositDeadline {
+                    if(block.timestamp < depositDeadline + lockDuration) {
+              }
+          }

```

### [G-24] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses ```OR/AND``` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
6 results - 2 files:
```solidity
src\LotterySetup.sol:

  54:         if (
  55:             lotterySetupParams.expectedPayout < lotterySetupParams.ticketPrice / 100
  56:                 || lotterySetupParams.expectedPayout >= lotterySetupParams.ticketPrice
  57:         ) {

  60          if (
  61:             lotterySetupParams.selectionSize > 16 || lotterySetupParams.selectionSize >= lotterySetupParams.selectionMax
  62          ) {

  65:         if (
  66:             lotterySetupParams.drawSchedule.drawCoolDownPeriod >= lotterySetupParams.drawSchedule.drawPeriod
  67:                 || lotterySetupParams.drawSchedule.firstDrawScheduledAt < lotterySetupParams.drawSchedule.drawPeriod
  68:         ) {

  123:         } else if (winTier == 0 || winTier > selectionSize) {
  124              return 0;

  165:         if (rewards.length != (selectionSize) || rewards[0] != 0) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L54-L57


```solidity
src\RNSourceController.sol:
   95:         if (notEnoughRetryInvocations || notEnoughTimeReachingMaxFailedAttempts) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L95

### [G-25] Optimize names to save gas (22 gas)

Contracts most called functions could simply save gas by function ordering via ```Method ID```. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because ```22 gas``` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

**Context:** 
All Contracts


**Recommendation:** 
Find a lower ```method ID``` name for the most called functions for example Call() vs. Call1() is cheaper by ```22 gas```
For example, the function IDs in the ```Lottery.sol``` contract will be the most used; A lower method ID may be given.

**Proof of Consept:**
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

Lottery.sol function names can be named and sorted according to METHOD ID

```js
Sighash   |   Function Signature
========================
67967759  =>  mintNativeTokens(address,uint256)
0b6e0961  =>  buyTickets(uint128[],uint120[],address,address)
2d49a7bf  =>  executeDraw()
451b19de  =>  unclaimedRewards(LotteryRewardType)
41f66c0c  =>  claimRewards(LotteryRewardType)
d1d58b25  =>  claimable(uint256)
df9dafab  =>  claimWinningTickets(uint256[])
f7f2ae99  =>  registerTicket(uint128,uint120,address,address)
054387ad  =>  receiveRandomNumber(uint256)
420d7990  =>  currentRewardSize(uint8)
e0f1985a  =>  drawRewardSize(uint128,uint8)
47da3f0f  =>  dueTicketsSoldAndReset(address)
ad25b575  =>  claimWinningTicket(uint256)
77959b4b  =>  returnUnclaimedJackpotToThePot()
b7f8dae2  =>  requireFinishedDraw(uint128)

```

### [G-26] Use ``< (>)`` instead of ``<= ( >=)``

There is no single opcode for ``≤`` or ``≥`` in Solidity. Solidity compiler executes LT/GT opcode and then executes an ISZERO opcode to check if the result of the previous comparison (LT/GT) is zero.

2 results - 2 files:
```solidity
src\Lottery.sol:

  272:    if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L272


```solidity
src\LotterySetup.sol:

   51:    if (lotterySetupParams.selectionMax >= 120) {

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L51


### [G-27] ``>=`` costs less gas than ``>``

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

2 results - 2 files:
```solidity
src\LotteryMath.sol:

  65:    excessPot = excessPotInt > 0 ? uint256(excessPotInt) : 0;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L65

```solidity
src\ReferralSystem.sol:

  158:   return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L158


### [G-28] Upgrade Solidity's optimizer

Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.

```js
lib\openzeppelin-contracts\hardhat.config.js:
  70  module.exports = {
  71:   solidity: {
  72:     version: argv.compiler,
  73:     settings: {
  74:       optimizer: {
  75:         enabled: withOptimizations,
  76:         runs: 200,
  77:       },

```