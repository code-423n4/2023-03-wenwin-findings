# Summary

## Gas Optimizations

|        | Issue                                                                                                                                 |
| ------ | :------------------------------------------------------------------------------------------------------------------------------------ |
| GAS-1  | `<X> += <Y>`/`<X> -= <Y>` COSTS MORE GAS THAN `<X> = <X> + <Y>`/`<X> = <X> - <Y>` FOR STATE VARIABLES                                 |
| GAS-2  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                                                           |
| GAS-3  | USING BOOLS FOR STORAGE INCURS OVERHEAD                                                                                               |
| GAS-4  | SETTING THE CONSTRUCTOR TO PAYABLE                                                                                                    |
| GAS-5  | DIV BY 0                                                                                                                              |
| GAS-6  | DOS WITH BLOCK GAS LIMIT                                                                                                              |
| GAS-7  | USE FUNCTION INSTEAD OF MODIFIERS                                                                                                     |
| GAS-8  | INTERNAL FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE REMOVED TO SAVE DEPLOYMENT GAS                                                |
| GAS-9  | INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS                                                                        |
| GAS-10 | MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT                                                                     |
| GAS-11 | MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL                                                                          |
| GAS-12 | MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE                    |
| GAS-13 | FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE                                                      |
| GAS-14 | OPTIMIZE NAMES TO SAVE GAS                                                                                                            |
| GAS-15 | PROPER DATA TYPES                                                                                                                     |
| GAS-16 | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD                                                       |
| GAS-17 | USE A MORE RECENT VERSION OF SOLIDITY                                                                                                 |
| GAS-18 | USE `REQUIRE` INSTEAD OF `ASSERT`                                                                                                     |
| GAS-19 | ADD `UNCHECKED {}` FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS `REQUIRE()`, `REVERT` OR `IF` STATEMENT |
| GAS-20 | USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS                                                                          |
| GAS-21 | USAGE OF `UINT`/`INT` SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD                                                                |
| GAS-22 | USE BYTES32 INSTEAD OF STRING                                                                                                         |
| GAS-23 | USE NAMED RETURNS WHERE APPROPRIATE                                                                                                   |

### [GAS-1] `<X> += <Y>`/`<X> -= <Y>` COSTS MORE GAS THAN `<X> = <X> + <Y>`/`<X> = <X> - <Y>` FOR STATE VARIABLES

#### Description:

Using the addition operator instead of plus-equals saves gas

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

129:         frontendDueTicketSales[frontend] += tickets.length;

173:             claimedAmount += claimWinningTicket(ticketIds[i]);

275:             currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);

```

```solidity
File: src/LotteryMath.sol

55:         newProfit -= int256(expectedRewardsOut);

64:         excessPotInt -= int256(fixedJackpotSize);

84:             bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);

```

```solidity
File: src/ReferralSystem.sol

64:                     totalTicketsForReferrersPerDraw[currentDraw] +=

67:                 totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;

69:             unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

71:         unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);

78:             claimedReward += claimPerDraw(drawIds[counter]);

147:             claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;

```

```solidity
File: src/TicketUtils.sol

29:                 ticketSize += (ticket & uint256(1));

96:                 winTier += uint8(intersection & uint120(1));

```

```solidity
File: src/staking/StakedTokenLock.sol

30:         depositedBalance += amount;

43:         depositedBalance -= amount;

```

### [GAS-2] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:

#### **Proof Of Concept**

```solidity
File: src/staking/StakedTokenLock.sol

34:         stakedToken.transferFrom(msg.sender, address(this), amount);

47:         stakedToken.transfer(msg.sender, amount);

55:         rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));

```

### [GAS-3] USING BOOLS FOR STORAGE INCURS OVERHEAD

#### Description:

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

#### **Proof Of Concept**

```solidity
File: src/RNSourceController.sol

16:     bool public override lastRequestFulfilled = true;

```

### [GAS-4] SETTING THE CONSTRUCTOR TO PAYABLE

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

84:     constructor(

```

```solidity
File: src/LotterySetup.sol

41:     constructor(LotterySetupParams memory lotterySetupParams) {

```

```solidity
File: src/LotteryToken.sol

17:     constructor() ERC20("Wenwin Lottery", "LOT") {

```

```solidity
File: src/RNSourceBase.sol

11:     constructor(address _authorizedConsumer) {

```

```solidity
File: src/RNSourceController.sol

26:     constructor(uint256 _maxFailedAttempts, uint256 _maxRequestDelay) {

```

```solidity
File: src/ReferralSystem.sol

27:     constructor(

```

```solidity
File: src/Ticket.sol

17:     constructor() ERC721("Wenwin Lottery Ticket", "WLT") { }

```

```solidity
File: src/VRFv2RNSource.sol

13:     constructor(

```

```solidity
File: src/staking/StakedTokenLock.sol

16:     constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {

```

```solidity
File: src/staking/Staking.sol

22:     constructor(

```

### [GAS-5] DIV BY 0

#### Description:

Division by 0 can lead to accidentally revert.

If expectedPayout is zero, then the division operation (excessPot _ EXCESS_BONUS_ALLOCATION) / (ticketsSold _ expectedPayout) will result in a division by zero error. Therefore, if expectedPayout is zero, the function will not execute the division operation and the bonusMulti value will remain at its initial value of PercentageMath.PERCENTAGE_BASE.

#### **Proof Of Concept**

```solidity
File: src/LotteryMath.sol

84:             bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);

```

### [GAS-6] DOS WITH BLOCK GAS LIMIT

#### Description:

When smart contracts are deployed or functions inside them are called, the execution of these actions always requires a certain amount of gas, based of how much computation is needed to complete them. The Ethereum network specifies a block gas limit and the sum of all transactions included in a block can not exceed the threshold.

Programming patterns that are harmless in centralized applications can lead to Denial of Service conditions in smart contracts when the cost of executing a function exceeds the block gas limit. Modifying an array of unknown size, that increases in size over time, can lead to such a Denial of Service condition.

Referecne: https://swcregistry.io/docs/SWC-128

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

125:         for (uint256 i = 0; i < drawIds.length; ++i) {

```

```solidity
File: src/ReferralSystem.sol

35:         for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {

77:         for (uint256 counter = 0; counter < drawIds.length; ++counter) {

```

#### Recommended Mitigation Steps:

Caution is advised when you expect to have large arrays that grow over time. Actions that require looping across the entire data structure should be avoided.

If you absolutely must loop over an array of unknown size, then you should plan for it to potentially take multiple blocks, and therefore require multiple transactions.

### [GAS-7] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

44:     modifier requireValidTicket(uint256 ticket) {

52:     modifier whenNotExecutingDraw() {

60:     modifier onlyWhenExecutingDraw() {

69:     modifier onlyTicketOwner(uint256 ticketId) {

```

```solidity
File: src/LotterySetup.sol

104:     modifier requireJackpotInitialized() {

112:     modifier beforeTicketRegistrationDeadline(uint128 drawId) {

```

### [GAS-8] INTERNAL FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE REMOVED TO SAVE DEPLOYMENT GAS

#### Description:

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

203:     function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {

279:     function requireFinishedDraw(uint128 drawId) internal view override {

285:     function mintNativeTokens(address mintTo, uint256 amount) internal override {

```

```solidity
File: src/PercentageMath.sol

17:     function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {

22:     function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {

```

```solidity
File: src/ReferralSystem.sol

87:     function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {

```

```solidity
File: src/Ticket.sol

19:     function markAsClaimed(uint256 ticketId) internal {

23:     function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {

```

```solidity
File: src/VRFv2RNSource.sol

28:     function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {

32:     function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {

```

```solidity
File: src/staking/Staking.sol

108:     function _beforeTokenTransfer(address from, address to, uint256) internal override {

```

### [GAS-9] INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

#### Description:

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

#### **Proof Of Concept**

```solidity
File: src/LotterySetup.sol

160:     function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {

```

```solidity
File: src/RNSourceController.sol

38:     function requestRandomNumber() internal {

```

```solidity
File: src/ReferralSystem.sol

156:     function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

161:     function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

```

### [GAS-10] MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple. There are four instances of public constants.

#### **Proof Of Concept**

```solidity
File: src/LotteryMath.sol

14:     uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;

16:     uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;

18:     uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;

20:     uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;

22:     uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;

24:     uint128 public constant DRAWS_PER_YEAR = 52;

```

```solidity
File: src/LotteryToken.sol

12:     uint256 public constant override INITIAL_SUPPLY = 1_000_000_000e18;

```

```solidity
File: src/PercentageMath.sol

8:     uint256 public constant PERCENTAGE_BASE = 100_000;

11:     uint256 public constant ONE_PERCENT = 1000;

```

### [GAS-11] MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

44:     modifier requireValidTicket(uint256 ticket) {

52:     modifier whenNotExecutingDraw() {

60:     modifier onlyWhenExecutingDraw() {

69:     modifier onlyTicketOwner(uint256 ticketId) {

```

```solidity
File: src/LotterySetup.sol

104:     modifier requireJackpotInitialized() {

112:     modifier beforeTicketRegistrationDeadline(uint128 drawId) {

```

### [GAS-12] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE

#### Description:

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.

If both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

27:     mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;

36:     mapping(uint128 => uint120) public override winningTicket;

37:     mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;

39:     mapping(uint128 => uint256) public override ticketsSold;

```

```solidity
File: src/ReferralSystem.sol

17:     mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;

19:     mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;

21:     mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;

23:     mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;

25:     mapping(uint128 => uint256) public override minimumEligibleReferrals;

```

```solidity
File: src/staking/Staking.sol

19:     mapping(address => uint256) public override userRewardPerTokenPaid;

20:     mapping(address => uint256) public override rewards;

```

#### Recommended Mitigation Steps:

Make mapping a struct instead

### [GAS-13] FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: src/RNSourceController.sol

77:     function initSource(IRNSource rnSource) external override onlyOwner {

89:     function swapSource(IRNSource newSource) external override onlyOwner {

```

```solidity
File: src/staking/StakedTokenLock.sol

24:     function deposit(uint256 amount) external override onlyOwner {

37:     function withdraw(uint256 amount) external override onlyOwner {

```

### [GAS-14] OPTIMIZE NAMES TO SAVE GAS

#### Description:

`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

[Solidity optimize name github](https://github.com/enzosv/solidity-optimize-name)

[Source](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

21: contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceController {

```

```solidity
File: src/LotterySetup.sol

12: contract LotterySetup is ILotterySetup {

```

```solidity
File: src/LotteryToken.sol

11: contract LotteryToken is ILotteryToken, ERC20 {

```

```solidity
File: src/RNSourceBase.sol

7: abstract contract RNSourceBase is IRNSource {

```

```solidity
File: src/RNSourceController.sol

10: abstract contract RNSourceController is Ownable2Step, IRNSourceController {

```

```solidity
File: src/ReferralSystem.sol

9: abstract contract ReferralSystem is IReferralSystem {


```

```solidity
File: src/Ticket.sol

12: abstract contract Ticket is ITicket, ERC721 {

```

```solidity
File: src/VRFv2RNSource.sol

9: contract VRFv2RNSource is IVRFv2RNSource, RNSourceBase, VRFV2WrapperConsumerBase {

```

```solidity
File: src/interfaces/ILottery.sol

50: interface ILottery is ITicket, ILotterySetup, IRNSourceController, IReferralSystem {

```

```solidity
File: src/interfaces/ILotterySetup.sol

80: interface ILotterySetup {

```

```solidity
File: src/interfaces/ILotteryToken.sol

11: interface ILotteryToken is IERC20 {

```

```solidity
File: src/interfaces/IRNSource.sol

5: interface IRNSource {

50: interface IRNSourceConsumer {

```

```solidity
File: src/interfaces/IRNSourceController.sol

36: interface IRNSourceController is IRNSourceConsumer {

```

```solidity
File: src/interfaces/IReferralSystem.sol

10: interface IReferralSystem {

```

```solidity
File: src/interfaces/IReferralSystemDynamic.sol

30: interface IReferralSystemDynamic {

```

```solidity
File: src/interfaces/ITicket.sol

9: interface ITicket is IERC721 {

```

```solidity
File: src/interfaces/IVRFv2RNSource.sol

12: interface IVRFv2RNSource is IRNSource {

```

```solidity
File: src/staking/StakedTokenLock.sol

9: contract StakedTokenLock is IStakedTokenLock, Ownable2Step {

```

```solidity
File: src/staking/Staking.sol

11: contract Staking is IStaking, ERC20 {

```

```solidity
File: src/staking/interfaces/IStakedTokenLock.sol

13: interface IStakedTokenLock {

```

```solidity
File: src/staking/interfaces/IStaking.sol

14: interface IStaking is IERC20 {

```

### [GAS-15] PROPER DATA TYPES

#### Description:

In Solidity, some data types are more expensive than others. It’s important to be aware of the most efficient type that can be used. Here are a few rules about data types.

Type uint should be used in place of type string whenever possible.

Type uint256 takes less gas to store than uint8

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

27:     mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;

36:     mapping(uint128 => uint120) public override winningTicket;

37:     mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;

39:     mapping(uint128 => uint256) public override ticketsSold;

111:         uint128[] calldata drawIds,

112:         uint120[] calldata tickets,

159:     function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier) {

162:             uint120 _winningTicket = winningTicket[ticketInfo.drawId];

182:         uint128 drawId,

183:         uint120 ticket,

204:         uint120 _winningTicket = TicketUtils.reconstructTicket(randomNumber, selectionSize, selectionMax);

205:         uint128 drawFinalized = currentDraw++;

211:             for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {

234:     function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {

238:     function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {

273:             uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;

279:     function requireFinishedDraw(uint128 drawId) internal view override {

```

```solidity
File: src/LotteryMath.sol

24:     uint128 public constant DRAWS_PER_YEAR = 52;

```

```solidity
File: src/LotterySetup.sol

30:     uint8 public immutable override selectionSize;

31:     uint8 public immutable override selectionMax;

112:     modifier beforeTicketRegistrationDeadline(uint128 drawId) {

120:     function fixedReward(uint8 winTier) public view override returns (uint256 amount) {

152:     function drawScheduledAt(uint128 drawId) public view override returns (uint256 time) {

156:     function ticketRegistrationDeadline(uint128 drawId) public view override returns (uint256 time) {

169:         for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {

170:             uint16 reward = uint16(rewards[winTier] / divisor);

```

```solidity
File: src/ReferralSystem.sol

17:     mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;

19:     mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;

21:     mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;

23:     mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;

25:     mapping(uint128 => uint256) public override minimumEligibleReferrals;

53:         uint128 currentDraw,

69:             unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

71:         unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);

76:     function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {

87:     function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {

134:     function requireFinishedDraw(uint128 drawId) internal view virtual;

136:     function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {

156:     function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

161:     function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

```

```solidity
File: src/Ticket.sol

23:     function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {

```

```solidity
File: src/TicketUtils.sol

19:         uint8 selectionSize,

20:         uint8 selectionMax

28:             for (uint8 i = 0; i < selectionMax; ++i) {

45:         uint8 selectionSize,

46:         uint8 selectionMax

50:         returns (uint120 ticket)

54:         uint8[] memory numbers = new uint8[](selectionSize);

58:             numbers[i] = uint8(randomNumber % currentSelectionCount);

66:             uint8 currentNumber = numbers[i];

74:             ticket |= ((uint120(1) << currentNumber));

84:         uint120 ticket,

85:         uint120 winningTicket,

86:         uint8 selectionSize,

87:         uint8 selectionMax

91:         returns (uint8 winTier)

94:             uint120 intersection = ticket & winningTicket;

95:             for (uint8 i = 0; i < selectionMax; ++i) {

96:                 winTier += uint8(intersection & uint120(1));

```

```solidity
File: src/VRFv2RNSource.sol

10:     uint16 public immutable override requestConfirmations;

11:     uint32 public immutable override callbackGasLimit;

17:         uint16 _requestConfirmations,

18:         uint32 _callbackGasLimit

```

```solidity
File: src/interfaces/ILottery.sol

60:         uint128 currentDraw,

62:         uint128 drawId,

64:         uint120 combination,

104:     function winAmount(uint128 drawId, uint8 winTier) external view returns (uint256 amount);

109:     function currentRewardSize(uint8 winTier) external view returns (uint256 rewardSize);

113:     function ticketsSold(uint128 drawId) external view returns (uint256 sold);

123:     function currentDraw() external view returns (uint128 drawId);

128:     function winningTicket(uint128 drawId) external view returns (uint120 winningCombination);

141:         uint128[] calldata drawIds,

142:         uint120[] calldata tickets,

165:     function claimable(uint256 ticketId) external view returns (uint256 claimableAmount, uint8 winTier);

```

```solidity
File: src/interfaces/ILotterySetup.sol

71:     uint8 selectionSize;

73:     uint8 selectionMax;

93:         uint8 indexed selectionSize,

94:         uint8 indexed selectionMax,

125:     function fixedReward(uint8 winTier) external view returns (uint256 amount);

132:     function selectionSize() external view returns (uint8 size);

137:     function selectionMax() external view returns (uint8 max);

154:     function drawScheduledAt(uint128 drawId) external view returns (uint256 time);

159:     function ticketRegistrationDeadline(uint128 drawId) external view returns (uint256 time);

```

```solidity
File: src/interfaces/IReferralSystem.sol

14:         uint128 referrerTicketCount;

16:         uint128 playerTicketCount;

30:         uint128 indexed drawId, uint256 indexed referrerRewardForDraw, uint256 indexed playerRewardForDraw

46:         uint128 drawId,

51:         returns (uint128 referrerTicketCount, uint128 playerTicketCount);

55:     function totalTicketsForReferrersPerDraw(uint128 drawId) external view returns (uint256 totalNumberOfTickets);

59:     function referrerRewardPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);

63:     function playerRewardsPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);

68:     function claimReferralReward(uint128[] memory drawIds) external returns (uint256 claimedReward);

73:     function minimumEligibleReferrals(uint128 drawId) external view returns (uint256 minimumEligibleReferrals);

```

```solidity
File: src/interfaces/ITicket.sol

13:         uint128 drawId;

15:         uint120 combination;

29:     function ticketsInfo(uint256 ticketId) external view returns (uint128 drawId, uint120 combination, bool claimed);

```

```solidity
File: src/interfaces/IVRFv2RNSource.sol

14:     function callbackGasLimit() external returns (uint32 gasLimit);

17:     function requestConfirmations() external returns (uint16 minConfirmations);

```

### [GAS-16] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

#### Description:

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

234:     function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {

```

```solidity
File: src/LotterySetup.sol

120:     function fixedReward(uint8 winTier) public view override returns (uint256 amount) {

```

### [GAS-17] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

Using at least `0.8.10` will save gas due to skipped extcodesize check if there is a return value. Currently the contracts are compiled using different versions `^0.8.7`, `^0.8.17`, `0.8.19`.

#### **Proof Of Concept**

```solidity
File: src/VRFv2RNSource.sol

3: pragma solidity ^0.8.7;

```

```solidity
File: src/interfaces/IVRFv2RNSource.sol

3: pragma solidity ^0.8.7;

```

```solidity
File: src/staking/StakedTokenLock.sol

3: pragma solidity ^0.8.17;

```

### [GAS-18] USE `REQUIRE` INSTEAD OF `ASSERT`

#### Description:

The `assert()` and `require()` functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the `assert()` function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a `require()` function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

#### **Proof Of Concept**

```solidity
File: src/LotterySetup.sol

147:         assert(initialPot > 0);

```

```solidity
File: src/TicketUtils.sol

99:             assert((winTier <= selectionSize) && (intersection == uint256(0)));

```

### [GAS-19] ADD `UNCHECKED {}` FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS `REQUIRE()`, `REVERT` OR `IF` STATEMENT

#### Description:

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block [see resource](https://github.com/ethereum/solidity/issues/10695)

`require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }`

`if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }`

This will stop the check for overflow and underflow so it will save gas

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

272:         if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {

273:                uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;

```

### [GAS-20] USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS

#### Description:

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

85:         LotterySetupParams memory lotterySetupParams,

88:         uint256[] memory rewardsToReferrersPerDraw,

119:         returns (uint256[] memory ticketIds)

160:         TicketInfo memory ticketInfo = ticketsInfo[ticketId];

```

```solidity
File: src/LotterySetup.sol

41:     constructor(LotterySetupParams memory lotterySetupParams) {

164:     function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {

```

```solidity
File: src/ReferralSystem.sol

30:         uint256[] memory _rewardsToReferrersPerDraw

76:     function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {

139:         UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];

```

```solidity
File: src/TicketUtils.sol

54:         uint8[] memory numbers = new uint8[](selectionSize);

63:         bool[] memory selected = new bool[](selectionMax);

```

```solidity
File: src/VRFv2RNSource.sol

32:     function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {

```

```solidity
File: src/interfaces/ILottery.sol

147:         returns (uint256[] memory ticketIds);

```

```solidity
File: src/interfaces/IReferralSystem.sol

68:     function claimReferralReward(uint128[] memory drawIds) external returns (uint256 claimedReward);

```

### [GAS-21] USAGE OF `UINT`/`INT` SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

Use a larger size then downcast where needed.

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

37:     mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;

159:     function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier) {

211:             for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {

234:     function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {

238:     function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {

```

```solidity
File: src/LotterySetup.sol

30:     uint8 public immutable override selectionSize;

31:     uint8 public immutable override selectionMax;

120:     function fixedReward(uint8 winTier) public view override returns (uint256 amount) {

126:             uint256 mask = uint256(type(uint16).max) << (winTier * 16);

169:         for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {

170:             uint16 reward = uint16(rewards[winTier] / divisor);

170:             uint16 reward = uint16(rewards[winTier] / divisor);

```

```solidity
File: src/TicketUtils.sol

19:         uint8 selectionSize,

20:         uint8 selectionMax

28:             for (uint8 i = 0; i < selectionMax; ++i) {

45:         uint8 selectionSize,

46:         uint8 selectionMax

54:         uint8[] memory numbers = new uint8[](selectionSize);

54:         uint8[] memory numbers = new uint8[](selectionSize);

58:             numbers[i] = uint8(randomNumber % currentSelectionCount);

66:             uint8 currentNumber = numbers[i];

86:         uint8 selectionSize,

87:         uint8 selectionMax

91:         returns (uint8 winTier)

95:             for (uint8 i = 0; i < selectionMax; ++i) {

96:                 winTier += uint8(intersection & uint120(1));

```

```solidity
File: src/VRFv2RNSource.sol

10:     uint16 public immutable override requestConfirmations;

11:     uint32 public immutable override callbackGasLimit;

17:         uint16 _requestConfirmations,

18:         uint32 _callbackGasLimit

```

```solidity
File: src/interfaces/ILottery.sol

104:     function winAmount(uint128 drawId, uint8 winTier) external view returns (uint256 amount);

109:     function currentRewardSize(uint8 winTier) external view returns (uint256 rewardSize);

165:     function claimable(uint256 ticketId) external view returns (uint256 claimableAmount, uint8 winTier);

```

```solidity
File: src/interfaces/ILotterySetup.sol

71:     uint8 selectionSize;

73:     uint8 selectionMax;

93:         uint8 indexed selectionSize,

94:         uint8 indexed selectionMax,

125:     function fixedReward(uint8 winTier) external view returns (uint256 amount);

132:     function selectionSize() external view returns (uint8 size);

137:     function selectionMax() external view returns (uint8 max);

```

```solidity
File: src/interfaces/IVRFv2RNSource.sol

14:     function callbackGasLimit() external returns (uint32 gasLimit);

17:     function requestConfirmations() external returns (uint16 minConfirmations);

```

### [GAS-22] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: src/RNSourceController.sol

114:         } catch Error(string memory reason) {

```

```solidity
File: src/staking/Staking.sol

26:         string memory name,

27:         string memory symbol

```

### [GAS-23] USE NAMED RETURNS WHERE APPROPRIATE

#### **Proof Of Concept**

```solidity
File: src/LotterySetup.sol

160:     function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {

```

```solidity
File: src/interfaces/ILottery.sol

98:     function drawExecutionInProgress() external view returns (bool);

```

```solidity
File: src/interfaces/IReferralSystem.sol

39:     function playerRewardFirstDraw() external view returns (uint256);

43:     function playerRewardDecreasePerDraw() external view returns (uint256);

```

```solidity
File: src/staking/interfaces/IStakedTokenLock.sol

15:     function stakedToken() external view returns (IStaking);

18:     function rewardsToken() external view returns (IERC20);

22:     function depositDeadline() external view returns (uint256);

25:     function lockDuration() external view returns (uint256);

28:     function depositedBalance() external view returns (uint256);

```

```solidity
File: src/staking/interfaces/IStaking.sol

30:     function rewardPerTokenStored() external view returns (uint256);

33:     function lastUpdateTicketId() external view returns (uint256);

36:     function userRewardPerTokenPaid(address account) external view returns (uint256);

39:     function rewards(address account) external view returns (uint256);

53:     function rewardsToken() external view returns (IERC20);

56:     function stakingToken() external view returns (IERC20);

```
