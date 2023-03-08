## Gas Optimizations
|  | Issue | Instances | Total Gas Saved |
| --- | --- | --- | --- |
| GAS-1 | Use assembly to check for `address(0)` | 10 | 60 |
| GAS-2 | `++i` costs less gas than `i++`, especially when it's used in `for` loops (`--i`/`i--` too) | 7 | - |
| GAS-3 | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables(`<x> -= <y>` too) | 3 | 339 |
| GAS-4 | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops | 11 | - |
| GAS-5 | Constructors can be marked `payable` | 10 | - |
| GAS-6 | Structs can be packed into fewer storage slots | 3 | 60000 |
| GAS-7 | Using `storage` instead of `memory` for structs/arrays saves gas | 2 | 4200 |
| GAS-8 | Using custom error instead of assert | 2 | - |
| GAS-9 | Perform all operations before emitting event | 1 | - |

Total: 49 instances over 9 issues with 64,599 gas saved
### [GAS-1] Use assembly to check for `address(0)`
Saves 6 gas per instance

Instances(10):
```
File: src/LotterySetup.sol

42:        if (address(lotterySetupParams.token) == address(0)) {{
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
File: src/ReferralSystem.sol

60:        if (referrer != address(0)) {
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
File: src/RNSourceController.sol

78:        if (address(rnSource) == address(0)) {

81:        if (address(source) != address(0)) {

90:        if (address(newSource) == address(0)) {
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol

```
File: src/staking/Staking.sol

31:        if (address(_lottery) == address(0)) {

34:        if (address(_rewardsToken) == address(0)) {

37:        if (address(_stakingToken) == address(0)) {

109:        if (from != address(0)) {

113:        if (to != address(0)) {
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol


### [Gas-2] `++i` costs less gas than `i++`, especially when it's used in `for` loops (`--i`/`i--` too)
Saves 5 gas per loop

Instances(7):
```
File: src/Lottery.sol

193:        unclaimedCount[drawId][ticket]++;

194:        ticketsSold[drawId]++;

205:        uint128 drawFinalized = currentDraw++;

266:        unclaimedCount[ticketsInfo[ticketId].drawId][ticketsInfo[ticketId].combination]--;
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
File: src/Ticket.sol

24:        ticketId = nextTicketId++;
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol

```
File: src/TicketUtils.sol

60:            currentSelectionCount--;

70:                    currentNumber++;
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

### [Gas-3] `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables(`<x> -= <y>` too)
Saves 113 gas per instance

Instances(3):
```
File: src/Lottery.sol

275:            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
File: src/staking/StakedTokenLock.sol

30:        depositedBalance += amount;

43:        depositedBalance -= amount;
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

### [Gas-4] `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops
Saves 30-40 gas per loop. [Source](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)

Instances(11):
```
File: src/Lottery.sol

125:        for (uint256 i = 0; i < drawIds.length; ++i) {

127:        for (uint256 i = 0; i < totalTickets; ++i) {

211:            for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
File: src/LotterySetup.sol

169:        for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
File: src/ReferralSystem.sol

35:        for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {

77:        for (uint256 counter = 0; counter < drawIds.length; ++counter) {
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
File: src/TicketUtils.sol

28:            for (uint8 i = 0; i < selectionMax; ++i) {

57:        for (uint256 i = 0; i < selectionSize; ++i) {

65:        for (uint256 i = 0; i < selectionSize; ++i) {

68:            for (uint256 j = 0; j <= currentNumber; ++j) {

95:            for (uint8 i = 0; i < selectionMax; ++i) {
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

### [Gas-5] Constructors can be marked `payable`
Payable functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided. A constructor can safely be marked as payable, since only the deployer would be able to pass funds.

Instances(10):
[src/Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol)
[src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)
[src/LotteryToken.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol)
[src/ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol)
[src/RNSourceBase.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol)
[src/RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol)
[src/Ticket.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol)
[src/VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol)
[src/staking/StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol)
[src/staking/Staking.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol)

### [Gas-6] Structs can be packed into fewer storage slots
Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings
Instances(3):
```
File: src/interfaces/ILotterySetup.sol

53: struct LotteryDrawSchedule {
54:     /// @dev First draw is scheduled to take place at this timestamp
55:     uint256 firstDrawScheduledAt;
56:     /// @dev Period for running lottery
57:     uint256 drawPeriod;
58:     /// @dev Cooldown period when users cannot register tickets for draw anymore
59:     uint256 drawCoolDownPeriod;
60: }
```
```
File: src/interfaces/ILotterySetup.sol

63: struct LotterySetupParams {
64:     /// @dev Token to be used as reward token for the lottery
65:     IERC20 token;
66:     /// @dev Parameters of the draw schedule for the lottery
67:     LotteryDrawSchedule drawSchedule;
68:     /// @dev Price to pay for playing single game (including fee)
69:     uint256 ticketPrice;
70 :    /// @dev Count of numbers user picks for the ticket
71:    uint8 selectionSize;
72:     /// @dev Max number user can pick
73:     uint8 selectionMax;
74:     /// @dev Expected payout for one ticket, expressed in `rewardToken`
75:     uint256 expectedPayout;
76:     /// @dev Array of fixed rewards per each non jackpot win
77:     uint256[] fixedRewards;
78: }
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol

```
File: src/interfaces/IReferralSystemDynamic.sol

24: struct MinimumReferralsRequirement {
25:     uint256 minimumTicketsSold;
26:     uint256 factor;
27:     ReferralRequirementFactorType factorType;
28: }
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystemDynamic.sol

### [Gas-7] Using `storage` instead of `memory` for structs/arrays saves gas
Use the `storage` instead of `memory` when declaring local variables to minimize gas consumption when fetching data from storage. Assigning the data to a memory variable reads all fields of the struct/array from storage, which incurs a 2100 gas for each field.

Instances(2):
```
File: src/Lottery.sol

160:        TicketInfo memory ticketInfo = ticketsInfo[ticketId];
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
File: src/ReferralSystem.sol

139:        UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

### [Gas-8]  Using custom error instead of assert
When the condition is false, `assert` tends to consume all remaining gas and undo all changes made.
But custom error refunds all remaining gas fees we offered to pay above and beyond, reverting all changes.

Instances(2):
```
File: src/LotterySetup.sol

147:        assert(initialPot > 0);
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
File: src/TicketUtils.sol

99:            assert((winTier <= selectionSize) && (intersection == uint256(0)));
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol


### [Gas-9] Perform all operations before emitting event
The following event performs some kind of storage access. However, should any of the activities coming after it reverts, the storage access will have been redundant. In some cases this equates to a wasted Gwarmaccess, costing 2100 gas per storage slot

Instances(1):
```
File: src/Lottery.sol

151:    function claimRewards(LotteryRewardType rewardType) external override returns (uint256 claimedAmount) {
152:        address beneficiary = (rewardType == LotteryRewardType.FRONTEND) ? msg.sender : stakingRewardRecipient;
153:        claimedAmount = LotteryMath.calculateRewards(ticketPrice, dueTicketsSoldAndReset(beneficiary), rewardType);
154:
155:        emit ClaimedRewards(beneficiary, claimedAmount, rewardType);
156:        rewardToken.safeTransfer(beneficiary, claimedAmount);
157:    }
```
Link to code: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol