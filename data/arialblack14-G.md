# GAS OPTIMIZATION REPORT

### Summary of optimizations


| Number     | Issue details                                                                                                 | Instances |
| ------------ | --------------------------------------------------------------------------------------------------------------- | :---------: |
| [G-1](#G1) | Using storage instead of memory for structs/arrays saves gas.                                                 |     2     |
| [G-2](#G2) | Usage of `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.                                      |    114    |
| [G-3](#G3) | Internal functions only called once can be inlined to save gas.                                               |     8     |
| [G-4](#G4) | `>=` costs less gas than `>`.                                                                                 |    19    |
| [G-5](#G5) | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables                                        |    16    |
| [G-6](#G6) | Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate. |     2     |
| [G-7](#G7) | Use assembly to check for address(0)                                                                          |    10    |

*Total: 7 issues.*

**Note: G1's instances differ from the ones in the automated findings output.**

---

### Gas Optimizations

### <a id=G1>[G-1]</a> Using storage instead of memory for structs/arrays saves gas.

##### Description

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.
Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct([src](https://ethereum.stackexchange.com/questions/128380/why-using-storage-keyword-instead-of-memory-cost-less-gas))

##### Recommendation

Use storage for `struct/array`

##### *Instances (2):*

File: [2023-03-wenwin/src/TicketUtils.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L54 )

```solidity
54: uint8[] memory numbers = new uint8[](selectionSize);
63: bool[] memory selected = new bool[](selectionMax);
```

### <a id=G2>[G-2]</a> Usage of `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.

##### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

##### Recommendation

Use `uint256` instead.

##### *Instances (114):*

File: [2023-03-wenwin/src/Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L27 )

```solidity
27: mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
34: uint128 public override currentDraw;
36: mapping(uint128 => uint120) public override winningTicket;
37: mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
39: mapping(uint128 => uint256) public override ticketsSold;
111: uint128[] calldata drawIds,
112: uint120[] calldata tickets,
159: function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier) {
162: uint120 _winningTicket = winningTicket[ticketInfo.drawId];
180: /// @param ticket Combination packed as uint120.
182: uint128 drawId,
183: uint120 ticket,
204: uint120 _winningTicket = TicketUtils.reconstructTicket(randomNumber, selectionSize, selectionMax);
205: uint128 drawFinalized = currentDraw++;
211: for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {
234: function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
238: function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {
273: uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
279: function requireFinishedDraw(uint128 drawId) internal view override {
```

File: [2023-03-wenwin/src/LotteryMath.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L24 )

```solidity
24: uint128 public constant DRAWS_PER_YEAR = 52;
```

File: [2023-03-wenwin/src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L30 )

```solidity
30: uint8 public immutable override selectionSize;
31: uint8 public immutable override selectionMax;
112: modifier beforeTicketRegistrationDeadline(uint128 drawId) {
120: function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
126: uint256 mask = uint256(type(uint16).max) << (winTier * 16);
152: function drawScheduledAt(uint128 drawId) public view override returns (uint256 time) {
156: function ticketRegistrationDeadline(uint128 drawId) public view override returns (uint256 time) {
169: for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {
170: uint16 reward = uint16(rewards[winTier] / divisor);
```

File: [2023-03-wenwin/src/ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L17 )

```solidity
17: mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;
19: mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;
21: mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;
23: mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;
25: mapping(uint128 => uint256) public override minimumEligibleReferrals;
53: uint128 currentDraw,
69: unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
71: unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
76: function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {
87: function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {
134: function requireFinishedDraw(uint128 drawId) internal view virtual;
136: function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

File: [2023-03-wenwin/src/Ticket.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L23 )

```solidity
23: function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
```

File: [2023-03-wenwin/src/TicketUtils.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L6 )

```solidity
6: /// Ticket is represented as uint120 packed ticket:
13: /// @param ticket Ticked represented as packed uint120
19: uint8 selectionSize,
20: uint8 selectionMax
28: for (uint8 i = 0; i < selectionMax; ++i) {
42: /// @return ticket Resulting ticket, packed as uint120
45: uint8 selectionSize,
46: uint8 selectionMax
50: returns (uint120 ticket)
54: uint8[] memory numbers = new uint8[](selectionSize);
58: numbers[i] = uint8(randomNumber % currentSelectionCount);
66: uint8 currentNumber = numbers[i];
74: ticket |= ((uint120(1) << currentNumber));
84: uint120 ticket,
85: uint120 winningTicket,
86: uint8 selectionSize,
87: uint8 selectionMax
91: returns (uint8 winTier)
94: uint120 intersection = ticket & winningTicket;
95: for (uint8 i = 0; i < selectionMax; ++i) {
96: winTier += uint8(intersection & uint120(1));
```

File: [2023-03-wenwin/src/VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L10 )

```solidity
10: uint16 public immutable override requestConfirmations;
11: uint32 public immutable override callbackGasLimit;
17: uint16 _requestConfirmations,
18: uint32 _callbackGasLimit
```

File: [2023-03-wenwin/src/interfaces/ILottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol#L39 )

```solidity
39: error DrawNotFinished(uint128 drawId);
56: /// @param combination Ticket combination represented as packed uint120.
60: uint128 currentDraw,
62: uint128 drawId,
64: uint120 combination,
83: event StartedExecutingDraw(uint128 indexed drawId);
88: /// @param winningTicket Winning ticket represented as packed uint120.
89: event FinishedExecutingDraw(uint128 indexed drawId, uint256 indexed randomNumber, uint120 indexed winningTicket);
104: function winAmount(uint128 drawId, uint8 winTier) external view returns (uint256 amount);
109: function currentRewardSize(uint8 winTier) external view returns (uint256 rewardSize);
113: function ticketsSold(uint128 drawId) external view returns (uint256 sold);
123: function currentDraw() external view returns (uint128 drawId);
128: function winningTicket(uint128 drawId) external view returns (uint120 winningCombination);
136: /// @param tickets list of uint120 packed tickets. Needs to be of same length as `drawIds`.
141: uint128[] calldata drawIds,
142: uint120[] calldata tickets,
165: function claimable(uint256 ticketId) external view returns (uint256 claimableAmount, uint8 winTier);
```

File: [2023-03-wenwin/src/interfaces/ILotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L50 )

```solidity
50: error TicketRegistrationClosed(uint128 drawId);
71: uint8 selectionSize;
73: uint8 selectionMax;
93: uint8 indexed selectionSize,
94: uint8 indexed selectionMax,
125: function fixedReward(uint8 winTier) external view returns (uint256 amount);
132: function selectionSize() external view returns (uint8 size);
137: function selectionMax() external view returns (uint8 max);
154: function drawScheduledAt(uint128 drawId) external view returns (uint256 time);
159: function ticketRegistrationDeadline(uint128 drawId) external view returns (uint256 time);
```

File: [2023-03-wenwin/src/interfaces/IReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L14 )

```solidity
14: uint128 referrerTicketCount;
16: uint128 playerTicketCount;
23: event ClaimedReferralReward(uint128 indexed drawId, address indexed user, uint256 indexed claimedAmount);
30: uint128 indexed drawId, uint256 indexed referrerRewardForDraw, uint256 indexed playerRewardForDraw
46: uint128 drawId,
51: returns (uint128 referrerTicketCount, uint128 playerTicketCount);
55: function totalTicketsForReferrersPerDraw(uint128 drawId) external view returns (uint256 totalNumberOfTickets);
59: function referrerRewardPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);
63: function playerRewardsPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);
68: function claimReferralReward(uint128[] memory drawIds) external returns (uint256 claimedReward);
73: function minimumEligibleReferrals(uint128 drawId) external view returns (uint256 minimumEligibleReferrals);
```

File: [2023-03-wenwin/src/interfaces/ITicket.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ITicket.sol#L13 )

```solidity
13: uint128 drawId;
14: /// @dev Ticket combination that is packed as uint120.
15: uint120 combination;
27: /// @return combination Ticket combination that is packed as uint120.
29: function ticketsInfo(uint256 ticketId) external view returns (uint128 drawId, uint120 combination, bool claimed);
```

File: [2023-03-wenwin/src/interfaces/IVRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L14 )

```solidity
14: function callbackGasLimit() external returns (uint32 gasLimit);
17: function requestConfirmations() external returns (uint16 minConfirmations);
```

### <a id=G3>[G-3]</a> Internal functions only called once can be inlined to save gas.

##### Description

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.
Check this out: [link](https://betterprogramming.pub/solidity-gas-optimizations-and-tricks-2bcee0f9f1f2)
Note: To sum up, when there is an internal function that is only called once, there is no point for that function to exist.

##### Recommendation

Remove the function and make it inline if it is a simple function.

##### *Instances (8):*

File: [2023-03-wenwin/src/LotteryMath.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L62 )

```solidity
62: function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot) {
```

File: [2023-03-wenwin/src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L160 )

```solidity
160: function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {
```

File: [2023-03-wenwin/src/PercentageMath.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L22 )

```solidity
22: function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
```

File: [2023-03-wenwin/src/ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L87 )

```solidity
87: function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {
156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

File: [2023-03-wenwin/src/Ticket.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L19 )

```solidity
19: function markAsClaimed(uint256 ticketId) internal {
```

File: [2023-03-wenwin/src/VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L28 )

```solidity
32: function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {
```

### <a id=G4>[G-4]</a> `>=` costs less gas than `>`.

##### Description

The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, which [saves 3 gas](https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde)

##### Recommendation

Use `>=` instead if appropriate.

##### *Instances (19):*

File: [2023-03-wenwin/src/Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L208 )

```solidity
208: if (jackpotWinners > 0) {
220: jackpotWinners > 0,
```

File: [2023-03-wenwin/src/LotteryMath.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L65 )

```solidity
65: excessPot = excessPotInt > 0 ? uint256(excessPotInt) : 0;
83: if (excessPot > 0 && ticketsSold > 0) {
```

File: [2023-03-wenwin/src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L61 )

```solidity
61: lotterySetupParams.selectionSize > 16 || lotterySetupParams.selectionSize >= lotterySetupParams.selectionMax
114: if (block.timestamp > ticketRegistrationDeadline(drawId)) {
123: } else if (winTier == 0 || winTier > selectionSize) {
133: if (initialPot > 0) {
147: assert(initialPot > 0);
```

File: [2023-03-wenwin/src/RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L27 )

```solidity
27: if (_maxFailedAttempts > MAX_MAX_FAILED_ATTEMPTS) {
30: if (_maxRequestDelay > MAX_REQUEST_DELAY) {
```

File: [2023-03-wenwin/src/ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L98 )

```solidity
98: if (totalTicketsForReferrersPerCurrentDraw > 0) {
104: if (playerRewardForDraw > 0) {
146: if (_unclaimedTickets.playerTicketCount > 0) {
151: if (claimedReward > 0) {
158: return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;
```

File: [2023-03-wenwin/src/staking/StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L26 )

```solidity
26: if (block.timestamp > depositDeadline) {
39: if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {
```

File: [2023-03-wenwin/src/staking/Staking.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L94 )

```solidity
94: if (reward > 0) {
```

### <a id=G5>[G-5]</a> `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

##### Description

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

##### Recommendation

Use `<x> = <x> + <y>` or `<x> = <x> - <y>` instead.

##### *Instances (16):*

File: [2023-03-wenwin/src/Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L129 )

```solidity
129: frontendDueTicketSales[frontend] += tickets.length;
173: claimedAmount += claimWinningTicket(ticketIds[i]);
275: currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

File: [2023-03-wenwin/src/LotteryMath.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L55 )

```solidity
55: newProfit -= int256(expectedRewardsOut);
64: excessPotInt -= int256(fixedJackpotSize);
84: bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
```

File: [2023-03-wenwin/src/ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L64 )

```solidity
64: totalTicketsForReferrersPerDraw[currentDraw] +=
67: totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
69: unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
71: unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
78: claimedReward += claimPerDraw(drawIds[counter]);
147: claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
```

File: [2023-03-wenwin/src/TicketUtils.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L29 )

```solidity
29: ticketSize += (ticket & uint256(1));
96: winTier += uint8(intersection & uint120(1));
```

File: [2023-03-wenwin/src/staking/StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30 )

```solidity
30: depositedBalance += amount;
43: depositedBalance -= amount;
```



### <a id=G6>[G-6]</a> Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate.

##### Description

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

```Solidity
mapping(address => uint256) public nonces;
-mapping (address => bool) public minters;
-mapping (address => bool) public markets;
+mapping (address => uint256) public markets_minters;
+            // address => uint256 => 1 (minters true)
+            // address => uint256 => 2 (minters false)
+            // address => uint256 => 3 (markets true)
+            // address => uint256 => 4 (markets false)```


##### Recommendation
Where appropriate, you can combine multiple address mappings into a single mapping of an address to a struct.
```

##### *Instances (2):*

File: [2023-03-wenwin/src/Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L27 )

```solidity
27: mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
37: mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
```

### <a id=G7>[G-7]</a> Use assembly to check for address(0)

##### Description

You can save about 6 gas per instance if using assembly to check for address(0)

##### Recommendation

Use assembly.

##### *Instances (10):*

File: [2023-03-wenwin/src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L42 )

```solidity
42: if (address(lotterySetupParams.token) == address(0)) {
```

File: [2023-03-wenwin/src/RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L78 )

```solidity
78: if (address(rnSource) == address(0)) {
81: if (address(source) != address(0)) {
90: if (address(newSource) == address(0)) {
```

File: [2023-03-wenwin/src/ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L60 )

```solidity
60: if (referrer != address(0)) {
```

File: [2023-03-wenwin/src/staking/Staking.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L31 )

```solidity
31: if (address(_lottery) == address(0)) {
34: if (address(_rewardsToken) == address(0)) {
37: if (address(_stakingToken) == address(0)) {
109: if (from != address(0)) {
113: if (to != address(0)) {
```

