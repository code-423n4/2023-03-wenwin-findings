## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | Use calldata instead of memory for function parameters | 2 | 600 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Setting the `constructor` to `payable` | 10 | 130 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Do not calculate constants | 6 | - |
| [GAS&#x2011;4](#GAS&#x2011;4) | Using `delete` statement can save gas | 8 | - |
| [GAS&#x2011;5](#GAS&#x2011;5) | Functions guaranteed to revert when called by normal users can be marked `payable` | 4 | 84 |
| [GAS&#x2011;6](#GAS&#x2011;6) | Use hardcoded address instead `address(this)` | 5 | - |
| [GAS&#x2011;7](#GAS&#x2011;7) | `internal` functions only called once can be inlined to save gas | 4 | 88 |
| [GAS&#x2011;8](#GAS&#x2011;8) | Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate | 2 | - |
| [GAS&#x2011;9](#GAS&#x2011;9) | Optimize names to save gas | 10 | 220 |
| [GAS&#x2011;10](#GAS&#x2011;10) | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 16 | - |
| [GAS&#x2011;11](#GAS&#x2011;11) | Public Functions To External | 8 | - |
| [GAS&#x2011;12](#GAS&#x2011;12) | `require()` Should Be Used Instead Of `assert()` | 2 | - |
| [GAS&#x2011;13](#GAS&#x2011;13) | Shorten the array rather than copying to a new one | 2 | - |
| [GAS&#x2011;14](#GAS&#x2011;14) | Help The Optimizer By Saving A Storage Variable’s Reference Instead Of Repeatedly Fetching It | 3 | - |
| [GAS&#x2011;15](#GAS&#x2011;15) | Structs can be packed into fewer storage slots | 2 | 4000 |
| [GAS&#x2011;16](#GAS&#x2011;16) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 21 | - |
| [GAS&#x2011;17](#GAS&#x2011;17) | Unnecessary look up in `if` condition | 1 | 2100 |
| [GAS&#x2011;18](#GAS&#x2011;18) | Use solidity version 0.8.19 to gain some gas boost | 3 | 264 |

Total: 109 contexts over 18 issues

## Gas Optimizations


### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Use calldata instead of memory for function parameters

In some cases, having function arguments in calldata instead of
memory is more optimal.

Consider the following generic example:
```
contract C {
    function add(uint[] memory arr) external returns (uint sum) {
        uint length = arr.length;
        for (uint i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
    }
}
```
In the above example, the dynamic array arr has the storage location
memory. When the function gets called externally, the array values are
kept in calldata and copied to memory during ABI decoding (using the
opcode calldataload and mstore). And during the for loop, arr[i]
accesses the value in memory using a mload. However, for the above
example this is inefficient. Consider the following snippet instead:
```
contract C {
    function add(uint[] calldata arr) external returns (uint sum) {
        uint length = arr.length;
        for (uint i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
    }
}
```
In the above snippet, instead of going via memory, the value is directly
read from calldata using calldataload. That is, there are no
intermediate memory operations that carries this value.

Gas savings: In the former example, the ABI decoding begins with
copying value from calldata to memory in a for loop. Each iteration
would cost at least 60 gas. In the latter example, this can be
completely avoided. This will also reduce the number of instructions and
therefore reduces the deploy time cost of the contract.

In short, use calldata instead of memory if the function argument
is only read.

Note that in older Solidity versions, changing some function arguments
from memory to calldata may cause "unimplemented feature error".
This can be avoided by using a newer (0.8.*) Solidity compiler.

Examples
Note: The following pattern is prevalent in the codebase:
```
function f(bytes memory data) external {
    (...) = abi.decode(data, (..., types, ...));
}
```
Here, changing to bytes calldata will decrease the gas. The total
savings for this change across all such uses would be quite
significant.

#### <ins>Proof Of Concept</ins>


```solidity
function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L164

```solidity
function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L32






### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

#### <ins>Proof Of Concept</ins>

```solidity
84: constructor(
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
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L84

```solidity
41: constructor(LotterySetupParams memory lotterySetupParams)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L41

```solidity
17: constructor() ERC20("Wenwin Lottery", "LOT")
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryToken.sol#L17

```solidity
27: constructor(
        uint256 _playerRewardFirstDraw,
        uint256 _playerRewardDecreasePerDraw,
        uint256[] memory _rewardsToReferrersPerDraw
    )
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L27

```solidity
11: constructor(address _authorizedConsumer)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceBase.sol#L11

```solidity
26: constructor(uint256 _maxFailedAttempts, uint256 _maxRequestDelay)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L26

```solidity
17: constructor() ERC721("Wenwin Lottery Ticket", "WLT")
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol#L17

```solidity
13: constructor(
        address _authorizedConsumer,
        address _linkAddress,
        address _wrapperAddress,
        uint16 _requestConfirmations,
        uint32 _callbackGasLimit
    )
        RNSourceBase(_authorizedConsumer)
        VRFV2WrapperConsumerBase(_linkAddress, _wrapperAddress)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L13

```solidity
16: constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L16

```solidity
22: constructor(
        ILottery _lottery,
        IERC20 _rewardsToken,
        IERC20 _stakingToken,
        string memory name,
        string memory symbol
    )
        ERC20(name, symbol)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L22





### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

#### <ins>Proof Of Concept</ins>

```solidity
14: uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol#L14

```solidity
16: uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol#L16

```solidity
18: uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol#L18

```solidity
20: uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol#L20

```solidity
22: uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol#L22

```solidity
36: uint256 private constant BASE_JACKPOT_PERCENTAGE = 30_030; // 30.03%
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L36





### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

```solidity
255: frontendDueTicketSales[beneficiary] = 0;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L255

```solidity
142: unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
148: unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L142

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L148



```solidity
52: failedSequentialAttempts = 0;
53: maxFailedAttemptsReachedAt = 0;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L52-L53

```solidity
99: failedSequentialAttempts = 0;
100: maxFailedAttemptsReachedAt = 0;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L99-L100

```solidity
95: rewards[msg.sender] = 0;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L95





### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### <ins>Proof Of Concept</ins>

```solidity
77: function initSource(IRNSource rnSource) external override onlyOwner {

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L77

```solidity
89: function swapSource(IRNSource newSource) external override onlyOwner {

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L89

```solidity
24: function deposit(uint256 amount) external override onlyOwner {

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L24

```solidity
37: function withdraw(uint256 amount) external override onlyOwner {

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L37



#### <ins>Recommended Mitigation Steps</ins>
Functions guaranteed to revert when called by normal users can be marked payable.



### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry's script.sol and solmate's `LibRlp.sol` contracts can help achieve this.

References: 
https://book.getfoundry.sh/reference/forge-std/compute-create-address 

https://twitter.com/transmissions11/status/1518507047943245824

#### <ins>Proof Of Concept</ins>

```solidity
130: rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L130

```solidity
140: uint256 raised = rewardToken.balanceOf(address(this));

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L140

```solidity
34: stakedToken.transferFrom(msg.sender, address(this), amount);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L34

```solidity
55: rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L55

```solidity
73: stakingToken.safeTransferFrom(msg.sender, address(this), amount);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L73



#### <ins>Recommended Mitigation Steps</ins>

Use hardcoded `address`






### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> `internal` functions only called once can be inlined to save gas

#### <ins>Proof Of Concept</ins>

```solidity
203: function receiveRandomNumber

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L203

```solidity
279: function requireFinishedDraw

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L279

```solidity
285: function mintNativeTokens

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L285

```solidity
160: function _baseJackpot

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L160




### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.

#### <ins>Proof Of Concept</ins>


```solidity
19: mapping(address => uint256) public override userRewardPerTokenPaid;
20: mapping(address => uint256) public override rewards;

//@audit Can be edited to the following struct:
struct addressRewardsStruct {
    uint256 userRewardPerTokenPaid;
    uint256 rewards;
}
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L20






### <a href="#Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

All in-scope contracts

#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.



### <a href="#Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

#### <ins>Proof Of Concept</ins>


```solidity
129: frontendDueTicketSales[frontend] += tickets.length;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L129

```solidity
173: claimedAmount += claimWinningTicket(ticketIds[i]);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L173

```solidity
275: currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L275

```solidity
55: newProfit -= int256(expectedRewardsOut);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol#L55

```solidity
64: excessPotInt -= int256(fixedJackpotSize);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol#L64

```solidity
84: bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol#L84

```solidity
64: totalTicketsForReferrersPerDraw[currentDraw] +=
67: totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
69: unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
71: unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L64

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L67

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L69

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L71



```solidity
78: claimedReward += claimPerDraw(drawIds[counter]);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L78

```solidity
147: claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L147

```solidity
29: ticketSize += (ticket & uint256(1));

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L29

```solidity
96: winTier += uint8(intersection & uint120(1));

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L96

```solidity
30: depositedBalance += amount;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L30

```solidity
43: depositedBalance -= amount;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L43







### <a href="#Summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L234

```solidity
function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L120

```solidity
function drawScheduledAt(uint128 drawId) public view override returns (uint256 time) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L152

```solidity
function ticketRegistrationDeadline(uint128 drawId) public view override returns (uint256 time) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L156

```solidity
function rewardPerToken() public view override returns (uint256 _rewardPerToken) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L48

```solidity
function earned(address account) public view override returns (uint256 _earned) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L61

```solidity
function withdraw(uint256 amount) public override {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L79

```solidity
function getReward() public override {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L91






### <a href="#Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> `require()` Should Be Used Instead Of `assert()`

#### <ins>Proof Of Concept</ins>


```solidity
147: assert(initialPot > 0);
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L147

```solidity
99: assert((winTier <= selectionSize) && (intersection == uint256(0)));
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L99





### <a href="#Summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

#### <ins>Proof Of Concept</ins>


```solidity
54: uint8[] memory numbers = new uint8[](selectionSize);
63: bool[] memory selected = new bool[](selectionMax);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L54

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L63








### <a href="#Summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> Help The Optimizer By Saving A Storage Variable's Reference Instead Of Repeatedly Fetching It

To help the optimizer, declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array.
The effect can be quite significant.
As an example, instead of repeatedly calling someMap[someIndex], save its reference like this: SomeStruct storage someStruct = someMap[someIndex] and use it.

#### <ins>Proof Of Concept</ins>



```solidity
170: uint16 reward = uint16(rewards[winTier] / divisor);
171: if ((rewards[winTier] % divisor) != 0) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L170-L171

```solidity
78: claimedReward += claimPerDraw(drawIds[counter]);
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L78






### <a href="#Summary">[GAS&#x2011;15]</a><a name="GAS&#x2011;15"> Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

#### <ins>Proof Of Concept</ins>

```solidity
63: struct LotterySetupParams {
    /// @dev Token to be used as reward token for the lottery
    IERC20 token;
    /// @dev Parameters of the draw schedule for the lottery
    LotteryDrawSchedule drawSchedule;
    /// @dev Price to pay for playing single game (including fee)
    uint256 ticketPrice;
    /// @dev Count of numbers user picks for the ticket
    uint8 selectionSize;
    /// @dev Max number user can pick
    uint8 selectionMax;
    /// @dev Expected payout for one ticket, expressed in `rewardToken`
    uint256 expectedPayout;
    /// @dev Array of fixed rewards per each non jackpot win
    uint256[] fixedRewards;
}

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L63

Can save 1 storage slot by changing to:

```solidity
63: struct LotterySetupParams {
    IERC20 token;
    uint8 selectionSize;
    uint8 selectionMax;
    LotteryDrawSchedule drawSchedule;
    uint256 ticketPrice;
    uint256 expectedPayout;
    uint256[] fixedRewards;
}

```

Can save an additional storage slot if `ticketPrice` and `expectedPayout` can be of size `uint128` if it is unlikely to ever reach `uint128.max` then we can save a total of 2 storage slots by changing to:

```solidity
63: struct LotterySetupParams {
    IERC20 token;
    uint8 selectionSize;
    uint8 selectionMax;
    LotteryDrawSchedule drawSchedule;
    uint128 ticketPrice;
    uint128 expectedPayout;
    uint256[] fixedRewards;
}

```







### <a href="#Summary">[GAS&#x2011;16]</a><a name="GAS&#x2011;16"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
34: uint128 public override currentDraw;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L34

```solidity
162: uint120 _winningTicket = winningTicket[ticketInfo.drawId];
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L162

```solidity
204: uint120 _winningTicket = TicketUtils.reconstructTicket(randomNumber, selectionSize, selectionMax);
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L204

```solidity
205: uint128 drawFinalized = currentDraw++;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L205

```solidity
273: uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L273

```solidity
24: uint128 public constant DRAWS_PER_YEAR = 52;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol#L24

```solidity
30: uint8 public immutable override selectionSize;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L30

```solidity
31: uint8 public immutable override selectionMax;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L31

```solidity
170: uint16 reward = uint16(rewards[winTier] / divisor);
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L170

```solidity
54: uint8[] memory numbers = new uint8[](selectionSize);
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L54

```solidity
66: uint8 currentNumber = numbers[i];
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L66

```solidity
94: uint120 intersection = ticket & winningTicket;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L94

```solidity
97: intersection >>= 1;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L97

```solidity
10: uint16 public immutable override requestConfirmations;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L10

```solidity
11: uint32 public immutable override callbackGasLimit;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L11

```solidity
71: uint8 selectionSize;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L71

```solidity
73: uint8 selectionMax;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L73

```solidity
14: uint128 referrerTicketCount;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L14

```solidity
16: uint128 playerTicketCount;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L16

```solidity
13: uint128 drawId;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ITicket.sol#L13

```solidity
15: uint120 combination;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ITicket.sol#L15







### <a href="#Summary">[GAS&#x2011;17]</a><a name="GAS&#x2011;17"> Unnecessary look up in `if` condition

If the `||` condition isn't required, the second condition will have been looked up unnecessarily.

#### <ins>Proof Of Concept</ins>

```solidity
95: if (notEnoughRetryInvocations || notEnoughTimeReachingMaxFailedAttempts) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L95






### <a href="#Summary">[GAS&#x2011;18]</a><a name="GAS&#x2011;18"> Use solidity version 0.8.19 to gain some gas boost

Upgrade to the latest solidity version 0.8.19 to get additional gas savings.
See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L3

```solidity
pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IVRFv2RNSource.sol#L3

```solidity
pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L3



