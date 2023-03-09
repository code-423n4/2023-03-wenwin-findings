# Gas Optimizations list

| Number | Details                                                                                           | Instances |
| ------ | ------------------------------------------------------------------------------------------------- | --------- |
| 1      | ```x += y```/```x -= y``` COSTS MORE GAS THAN ```x = x + y```/```x = x - y``` FOR STATE VARIABLES | 11        |
| 2      | CAN DECLARE VARIABLE OUTSIDE LOOP TO SAVE GAS                                                     | 2         |
| 3      | MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMNBINED INTO A SINGLE MAPPING TO A STRUCT                   | 10        |
| 4      | AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS                                          | 1         |
| 5      | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD                   | 2         |
| 6      | USE ```bytes32``` INSTEAD OF ```string``` WHEREVER POSSIBLE                                       | 1         |
| 7      | OPTIMIZE NAMES TO SAVE GAS                                                                        | 24        |
| 8      | INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS                                    | 5         |
| 9      | STATE VARIABLES CAN BE PACKED INTO FEWER STORAGE SLOTS                                            | 1         |
| 10     | ```revert()``` STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION         | 1         |
| 11     | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                       | 2         |
| 12     | REORDER STRUCTURE LAYOUT                                                                          | 1         |
| 13     | CACHE STORAGE VALUES IN MEMORY TO MINIMIZE SLOADS                                                 | 12        |
| 14     | SHOULD USE LOCAL ARGUMENTS INSTEAD OF STATE VARIABLE                                              | 1         |
| 15     | USE ASSEMBLY TO CHECK FOR ```address(0)```                                                        | 10        |
| 16     | USE ```require``` INSTEAD OF ```assert```                                                         | 2         |
| 17     | USE ```calldata``` INSTEAD OF ```memory```                                                        | 3         |

# Gas Optimizations
## [G-01]  ```x += y```/```x -= y``` COSTS MORE GAS THAN ```x = x + y```/```x = x - y``` FOR STATE VARIABLES
Using the addition operator instead of plus-equals saves some gas for state variables.

[src\Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol)
```js
129:         frontendDueTicketSales[frontend] += tickets.length;

173:             claimedAmount += claimWinningTicket(ticketIds[i]);

275:             currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

[src\ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol)
```js
64:                     totalTicketsForReferrersPerDraw[currentDraw] +=
65:                         unclaimedTickets[currentDraw][referrer].referrerTicketCount;

67:                 totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;

69:             unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

71:         unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);

78:             claimedReward += claimPerDraw(drawIds[counter]);

147:             claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
```

[src\staking\StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol)
```js
30:         depositedBalance += amount;

43:         depositedBalance -= amount;
```

## [G-02] CAN DECLARE VARIABLE OUTSIDE LOOP TO SAVE GAS  
Declaring the stack variable outside the loop will save gas that would otherwise be spent on declaring the variable over and over again.      

[src\LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)
```js
170:             uint16 reward = uint16(rewards[winTier] / divisor);
```

[src\TicketUtils.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol)
```js
66:             uint8 currentNumber = numbers[i];
```

## [G-03] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMNBINED INTO A SINGLE MAPPING TO A STRUCT
If both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Depending on the circumstances and sizes of types, can avoid a Gsset per mapping combined.

[src\Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol)
```js
27:     mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
36:     mapping(uint128 => uint120) public override winningTicket;
37:     mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
```

[src\ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol)
```js
17:     mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;
18: 
19:     mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;
20: 
21:     mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;
22: 
23:     mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;
24: 
25:     mapping(uint128 => uint256) public override minimumEligibleReferrals;
```

[src\staking\Staking.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol)
```js
19:     mapping(address => uint256) public override userRewardPerTokenPaid;
20:     mapping(address => uint256) public override rewards;
```

## [G-04] AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS
Before Solidity 0.8.10 the compiler inserted extra code to check for contract existence for external function calls ```EXTCODESIZE``` (100 gas). In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

[src\VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol)
```js
29:         requestId = requestRandomness(callbackGasLimit, requestConfirmations, 1);
```

## [G-05] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD
Contracts are allowed to override their parents’ functions and change the visibility from public to external and can save gas by doing so.

[src\Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol)
```js
234:     function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
```

[src\LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)
```js
120:     function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
```

## [G-06] USE ```bytes32``` INSTEAD OF ```string``` WHEREVER POSSIBLE
Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

[src\RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol)
```js
114:         } catch Error(string memory reason) {
```

## [G-07] OPTIMIZE NAMES TO SAVE GAS
Function hashes that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted. Prioritize the most called functions and sort and rename them according to the function hashes/method IDs. For a better understanding please refer to [this link](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

The method IDs in the Lottery.sol will be used the most. A lower method ID may be given to the most frequently used functions. [This](https://emn178.github.io/solidity-optimize-name/) is a useful tool that can be used for the same, if required. 

```json
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

## [G-08] INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS
Not inlining costs 20 to 40 gas because of two extra ```JUMP``` instructions and additional stack operations needed for function calls. 

[src\LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)
```js
160:     function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {
```

[src\ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol)
```js
111:     function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
112:         internal
113:         view
114:         virtual
115:         returns (uint256 minimumEligible)

156:     function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

161:     function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

[src\staking\Staking.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol)
```js
108:     function _beforeTokenTransfer(address from, address to, uint256) internal override {
```

## [G-09] STATE VARIABLES CAN BE PACKED INTO FEWER STORAGE SLOTS
If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

[src\RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol)
```js
11:     IRNSource public override source;
12: 
13:     uint256 public override failedSequentialAttempts;
14:     uint256 public override maxFailedAttemptsReachedAt;
15:     uint256 public override lastRequestTimestamp;
16:     bool public override lastRequestFulfilled = true;
```

## [G-10] ```revert()``` STATEMENTS THAT RESULT FROM CHECKING INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION
By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

[src\LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)
```js
170:             uint16 reward = uint16(rewards[winTier] / divisor);
171:             if ((rewards[winTier] % divisor) != 0) {
172:                 revert InvalidFixedRewardSetup();
173:             }
```

## [G-11] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE
Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything.

[src\staking\StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol)
```js
34:         stakedToken.transferFrom(msg.sender, address(this), amount);

47:         stakedToken.transfer(msg.sender, amount);
```

## [G-12] REORDER STRUCTURE LAYOUT
The following structs could be optimized by moving the position of certain variables in order to optimize storage. 

[src\interfaces\ILotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol)
```js
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

## [G-13] CACHE STORAGE VALUES IN MEMORY TO MINIMIZE SLOADS
SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

[src\Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol)
```js
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

272:         if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {
273:             uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
```

[src\LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)
```js
133:         if (initialPot > 0) {

147:         assert(initialPot > 0);
```

[src\ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol)
```js
62:             if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
63:                 if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
64:                     totalTicketsForReferrersPerDraw[currentDraw] +=
65:                         unclaimedTickets[currentDraw][referrer].referrerTicketCount;
66:                 }
67:                 totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
68:             }
69:             unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
```

[src\RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol)
```js
68:         uint256 failedAttempts = ++failedSequentialAttempts;
73:         emit Retry(source, failedSequentialAttempts);
```

## [G-14] SHOULD USE LOCAL ARGUMENTS INSTEAD OF STATE VARIABLE
Using the local arguments of the function while emitting an event instead of the state variable saves roughly 122 gas per emit.

[src\RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol)
```js
73:         emit Retry(source, failedSequentialAttempts);
```

## [G-15] USE ASSEMBLY TO CHECK FOR ```address(0)```
Saves 6 gas per instance if using assembly to check for address(0).

[src\LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)
```js
42:         if (address(lotterySetupParams.token) == address(0)) {
```

[src\ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol)
```js
60:         if (referrer != address(0)) {
```

[src\RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol):
```js
78:         if (address(rnSource) == address(0)) {

81:         if (address(source) != address(0)) {

90:         if (address(newSource) == address(0)) {
```

[src\staking\Staking.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol)
```js
31:         if (address(_lottery) == address(0)) {

34:         if (address(_rewardsToken) == address(0)) {

37:         if (address(_stakingToken) == address(0)) {

109:         if (from != address(0)) {

113:         if (to != address(0)) {
```

## [G-16] USE ```require``` INSTEAD OF ```assert```
assert() function when false, uses up all the remaining gas and reverts all the changes made.

[src\LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)
```js
147:         assert(initialPot > 0);
```

[src\TicketUtils.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol)
```js
99:             assert((winTier <= selectionSize) && (intersection == uint256(0)));
```

## [G-17] USE ```calldata``` INSTEAD OF ```memory```
When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

[src\LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)
```js
164:     function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {
```

[src\ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol)
```js
76:     function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {
```

[src\VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol)
```js
32:     function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {
```