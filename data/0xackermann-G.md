# Gas Report 

[Exclusive findings](#exclusive-findings) is below the [automated findings](#automated-findings).

---
## Gas Optimizations

| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Using bools for storage incurs overhead | 2 |
| [GAS-2](#GAS-2) | Cache array length outside of loop | 3 |
| [GAS-3](#GAS-3) | Don't initialize variables with default value | 9 |
| [GAS-4](#GAS-4) | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | 4 |
| [GAS-5](#GAS-5) | Using `private` rather than `public` for constants, saves gas | 9 |
| [GAS-6](#GAS-6) | Use shift Right/Left instead of division/multiplication if possible | 3 |
| [GAS-7](#GAS-7) | Incrementing with a smaller type than `uint256` incurs overhead | 5 |
| [GAS-8](#GAS-8) | Use `storage` instead of `memory` for structs/arrays | 4 |
| [GAS-9](#GAS-9) | Increments can be `unchecked` in for-loops | 11 |
| [GAS-10](#GAS-10) | Use != 0 instead of > 0 for unsigned integer comparison | 12 |
### <a name="GAS-1"></a>[GAS-1] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (2)*:
```solidity
File: src/Lottery.sol

33:     bool public override drawExecutionInProgress;

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol)

```solidity
File: src/RNSourceController.sol

16:     bool public override lastRequestFulfilled = true;

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol)

### <a name="GAS-2"></a>[GAS-2] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (3)*:
```solidity
File: src/Lottery.sol

125:         for (uint256 i = 0; i < drawIds.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol)

```solidity
File: src/ReferralSystem.sol

35:         for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {

77:         for (uint256 counter = 0; counter < drawIds.length; ++counter) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol)

### <a name="GAS-3"></a>[GAS-3] Don't initialize variables with default value

*Instances (9)*:
```solidity
File: src/Lottery.sol

125:         for (uint256 i = 0; i < drawIds.length; ++i) {

172:         for (uint256 i = 0; i < totalTickets; ++i) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol)

```solidity
File: src/ReferralSystem.sol

35:         for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {

77:         for (uint256 counter = 0; counter < drawIds.length; ++counter) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol)

```solidity
File: src/TicketUtils.sol

28:             for (uint8 i = 0; i < selectionMax; ++i) {

57:         for (uint256 i = 0; i < selectionSize; ++i) {

65:         for (uint256 i = 0; i < selectionSize; ++i) {

68:             for (uint256 j = 0; j <= currentNumber; ++j) {

95:             for (uint8 i = 0; i < selectionMax; ++i) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol)

### <a name="GAS-4"></a>[GAS-4] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
*Saves 5 gas per iteration*

*Instances (4)*:
```solidity
File: src/Lottery.sol

205:         uint128 drawFinalized = currentDraw++;

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol)

```solidity
File: src/Ticket.sol

24:         ticketId = nextTicketId++;

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol)

```solidity
File: src/TicketUtils.sol

60:             currentSelectionCount--;

70:                     currentNumber++;

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol)

### <a name="GAS-5"></a>[GAS-5] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (9)*:
```solidity
File: src/LotteryMath.sol

14:     uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;

16:     uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;

18:     uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;

20:     uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;

22:     uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;

24:     uint128 public constant DRAWS_PER_YEAR = 52;

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol)

```solidity
File: src/LotteryToken.sol

12:     uint256 public constant override INITIAL_SUPPLY = 1_000_000_000e18;

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryToken.sol)

```solidity
File: src/PercentageMath.sol

8:     uint256 public constant PERCENTAGE_BASE = 100_000;

11:     uint256 public constant ONE_PERCENT = 1000;

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/PercentageMath.sol)

### <a name="GAS-6"></a>[GAS-6] Use shift Right/Left instead of division/multiplication if possible
Shifting left by N is like multiplying by 2^N and shifting right by N is like dividing by 2^N

*Instances (3)*:
```solidity
File: src/LotterySetup.sol

127:             uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);

128:             return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));

175:         }

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol)

### <a name="GAS-7"></a>[GAS-7] Incrementing with a smaller type than `uint256` incurs overhead

*Instances (5)*:
```solidity
File: src/Lottery.sol

211:             for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol)

```solidity
File: src/LotterySetup.sol

112:     modifier beforeTicketRegistrationDeadline(uint128 drawId) {

169:         for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol)

```solidity
File: src/TicketUtils.sol

28:             for (uint8 i = 0; i < selectionMax; ++i) {

95:             for (uint8 i = 0; i < selectionMax; ++i) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol)

### <a name="GAS-8"></a>[GAS-8] Use `storage` instead of `memory` for structs/arrays
Using `memory` copies the struct or array in memory. Use `storage` to save the location in storage and have cheaper reads:

*Instances (4)*:
```solidity
File: src/Lottery.sol

161:         if (!ticketInfo.claimed) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol)

```solidity
File: src/ReferralSystem.sol

140:         if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol)

```solidity
File: src/TicketUtils.sol

55:         uint256 currentSelectionCount = uint256(selectionMax);

64: 

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol)

### <a name="GAS-9"></a>[GAS-9] Increments can be `unchecked` in for-loops

*Instances (11)*:
```solidity
File: src/Lottery.sol

125:         for (uint256 i = 0; i < drawIds.length; ++i) {

172:         for (uint256 i = 0; i < totalTickets; ++i) {

211:             for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol)

```solidity
File: src/LotterySetup.sol

169:         for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol)

```solidity
File: src/ReferralSystem.sol

35:         for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {

77:         for (uint256 counter = 0; counter < drawIds.length; ++counter) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol)

```solidity
File: src/TicketUtils.sol

28:             for (uint8 i = 0; i < selectionMax; ++i) {

57:         for (uint256 i = 0; i < selectionSize; ++i) {

65:         for (uint256 i = 0; i < selectionSize; ++i) {

68:             for (uint256 j = 0; j <= currentNumber; ++j) {

95:             for (uint8 i = 0; i < selectionMax; ++i) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol)

### <a name="GAS-10"></a>[GAS-10] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (12)*:
```solidity
File: src/Lottery.sol

208:         if (jackpotWinners > 0) {

220:             jackpotWinners > 0,

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol)

```solidity
File: src/LotteryMath.sol

65:         excessPot = excessPotInt > 0 ? uint256(excessPotInt) : 0;

83:         if (excessPot > 0 && ticketsSold > 0) {

83:         if (excessPot > 0 && ticketsSold > 0) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol)

```solidity
File: src/LotterySetup.sol

133:         if (initialPot > 0) {

147:         assert(initialPot > 0);

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol)

```solidity
File: src/ReferralSystem.sol

98:         if (totalTicketsForReferrersPerCurrentDraw > 0) {

104:         if (playerRewardForDraw > 0) {

146:         if (_unclaimedTickets.playerTicketCount > 0) {

151:         if (claimedReward > 0) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol)

```solidity
File: src/staking/Staking.sol

94:         if (reward > 0) {

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol)


---
## Exclusive Findings
---
### Gas Optimizations Report
| No. | Issue |
| --- | --- |
| 1 | [Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#G-01-usage-of-uintint-smaller-than-32-bytes-256-bits-incurs-overhead)|
| 2 | [++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#G-02-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for-loop-and-while-loops)|
| 3 | [Optimize names to save gas [22 gas per instance]](#G-03-optimize-names-to-save-gas-22-gas-per-instance)|
| 4 | [Setting the `constructor` to `payable` [~13 gas per instance]](#G-04-setting-the-constructor-to-payable-13-gas-per-instance)|
| 5 | [Comparison operators](#G-05-comparison-operators)|
| 6 | [Use a more recent version of solidity](#G-06-use-a-more-recent-version-of-solidity)|
| 7 | [`<array>.length` should not be looked up in every loop of a for-loop](#G-07-arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)|
| 8 | [`x += y` costs more gas than `x = x + y` for state variables](#G-08-x--y-costs-more-gas-than-x--x--y-for-state-variables)|
| 9 | [Multiple `address` mappings can be combined into a single `mapping` of an `address` to a `struct`, where appropriate](#G-09-multiple-address-mappings-can-be-combined-into-a-single-mapping-of-an-address-to-a-struct-where-appropriate)|
---
### [G-01] Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

---
**Description:**

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so.

**Recommendation:**

Use a larger size then downcast where needed

**Lines of Code:** 

- [Lottery.sol#L27](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L27)
- [Lottery.sol#L27](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L27)
- [Lottery.sol#L34](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L34)
- [Lottery.sol#L36](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L36)
- [Lottery.sol#L37](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L37)
- [Lottery.sol#L37](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L37)
- [Lottery.sol#L39](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L39)
- [Lottery.sol#L159](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L159)
- [Lottery.sol#L162](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L162)
- [Lottery.sol#L182](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L182)
- [Lottery.sol#L183](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L183)
- [Lottery.sol#L204](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L204)
- [Lottery.sol#L205](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L205)
- [Lottery.sol#L211](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L211)
- [Lottery.sol#L234](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L234)
- [Lottery.sol#L238](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L238)
- [Lottery.sol#L238](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L238)
- [Lottery.sol#L273](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L273)
- [Lottery.sol#L279](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L279)
- [LotteryMath.sol#L24](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol#L24)
- [LotterySetup.sol#L30](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L30)
- [LotterySetup.sol#L31](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L31)
- [LotterySetup.sol#L112](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L112)
- [LotterySetup.sol#L120](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L120)
- [LotterySetup.sol#L152](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L152)
- [LotterySetup.sol#L156](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L156)
- [LotterySetup.sol#L169](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L169)
- [LotterySetup.sol#L170](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L170)
- [ReferralSystem.sol#L17](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L17)
- [ReferralSystem.sol#L19](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L19)
- [ReferralSystem.sol#L21](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L21)
- [ReferralSystem.sol#L23](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L23)
- [ReferralSystem.sol#L25](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L25)
- [ReferralSystem.sol#L53](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L53)
- [ReferralSystem.sol#L87](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L87)
- [ReferralSystem.sol#L134](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L134)
- [ReferralSystem.sol#L136](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L136)
- [ReferralSystem.sol#L156](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L156)
- [ReferralSystem.sol#L161](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L161)
- [Ticket.sol#L23](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol#L23)
- [Ticket.sol#L23](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol#L23)
- [TicketUtils.sol#L19](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L19)
- [TicketUtils.sol#L20](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L20)
- [TicketUtils.sol#L28](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L28)
- [TicketUtils.sol#L45](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L45)
- [TicketUtils.sol#L46](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L46)
- [TicketUtils.sol#L50](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L50)
- [TicketUtils.sol#L66](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L66)
- [TicketUtils.sol#L84](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L84)
- [TicketUtils.sol#L85](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L85)
- [TicketUtils.sol#L86](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L86)
- [TicketUtils.sol#L87](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L87)
- [TicketUtils.sol#L91](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L91)
- [TicketUtils.sol#L94](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L94)
- [TicketUtils.sol#L95](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L95)
- [VRFv2RNSource.sol#L10](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L10)
- [VRFv2RNSource.sol#L11](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L11)
- [VRFv2RNSource.sol#L17](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L17)
- [VRFv2RNSource.sol#L18](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L18)
- [ILottery.sol#L39](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L39)
- [ILottery.sol#L60](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L60)
- [ILottery.sol#L62](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L62)
- [ILottery.sol#L64](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L64)
- [ILottery.sol#L83](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L83)
- [ILottery.sol#L89](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L89)
- [ILottery.sol#L89](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L89)
- [ILottery.sol#L104](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L104)
- [ILottery.sol#L104](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L104)
- [ILottery.sol#L109](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L109)
- [ILottery.sol#L113](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L113)
- [ILottery.sol#L123](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L123)
- [ILottery.sol#L128](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L128)
- [ILottery.sol#L128](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L128)
- [ILottery.sol#L165](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol#L165)
- [ILotterySetup.sol#L50](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L50)
- [ILotterySetup.sol#L71](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L71)
- [ILotterySetup.sol#L73](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L73)
- [ILotterySetup.sol#L93](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L93)
- [ILotterySetup.sol#L94](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L94)
- [ILotterySetup.sol#L125](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L125)
- [ILotterySetup.sol#L132](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L132)
- [ILotterySetup.sol#L137](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L137)
- [ILotterySetup.sol#L154](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L154)
- [ILotterySetup.sol#L159](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L159)
- [IReferralSystem.sol#L14](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L14)
- [IReferralSystem.sol#L16](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L16)
- [IReferralSystem.sol#L23](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L23)
- [IReferralSystem.sol#L30](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L30)
- [IReferralSystem.sol#L46](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L46)
- [IReferralSystem.sol#L51](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L51)
- [IReferralSystem.sol#L51](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L51)
- [IReferralSystem.sol#L55](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L55)
- [IReferralSystem.sol#L59](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L59)
- [IReferralSystem.sol#L63](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L63)
- [IReferralSystem.sol#L73](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol#L73)
- [ITicket.sol#L13](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ITicket.sol#L13)
- [ITicket.sol#L15](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ITicket.sol#L15)
- [ITicket.sol#L29](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ITicket.sol#L29)
- [ITicket.sol#L29](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ITicket.sol#L29)
- [IVRFv2RNSource.sol#L14](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IVRFv2RNSource.sol#L14)
- [IVRFv2RNSource.sol#L17](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IVRFv2RNSource.sol#L17)

---
### [G-02] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

---
**Description:**

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

**Recommendation:**

Using Solidity's unchecked block saves the overflow checks.

**Proof Of Concept:**

<https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g011---unnecessary-checked-arithmetic-in-for-loop>

**Lines of Code:** 

- [Lottery.sol#L125](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L125)
- [Lottery.sol#L172](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L172)
- [ReferralSystem.sol#L35](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L35)
- [TicketUtils.sol#L28](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L28)
- [TicketUtils.sol#L57](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L57)
- [TicketUtils.sol#L65](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L65)
- [TicketUtils.sol#L95](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L95)

---
### [G-03] Optimize names to save gas [22 gas per instance]

---
**Description:**

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

**Recommendation:**

Find a lower `method ID` name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas. For example, the function IDs in the L1GraphTokenGateway.sol contract will be the most used; A lower method ID may be given.

**Proof Of Concept:**

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

**Lines of Code:** 

- [Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol)
- [LotteryMath.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol)
- [LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol)
- [LotteryToken.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryToken.sol)
- [PercentageMath.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/PercentageMath.sol)
- [RNSourceBase.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceBase.sol)
- [RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol)
- [ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol)
- [Ticket.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol)
- [TicketUtils.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol)
- [VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol)
- [ILottery.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol)
- [ILotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol)
- [ILotteryToken.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotteryToken.sol)
- [IRNSource.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IRNSource.sol)
- [IRNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IRNSourceController.sol)
- [IReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol)
- [IReferralSystemDynamic.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystemDynamic.sol)
- [ITicket.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ITicket.sol)
- [IVRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IVRFv2RNSource.sol)
- [StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol)
- [Staking.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol)
- [IStakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/interfaces/IStakedTokenLock.sol)
- [IStaking.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/interfaces/IStaking.sol)

---
### [G-04] Setting the `constructor` to `payable` [~13 gas per instance]

---
**Lines of Code:** 

- [Lottery.sol#L84](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L84)
- [LotterySetup.sol#L41](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L41)
- [LotteryToken.sol#L17](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryToken.sol#L17)
- [RNSourceBase.sol#L11](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceBase.sol#L11)
- [RNSourceController.sol#L26](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L26)
- [ReferralSystem.sol#L27](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L27)
- [Ticket.sol#L17](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol#L17)
- [VRFv2RNSource.sol#L13](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L13)
- [StakedTokenLock.sol#L16](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L16)
- [Staking.sol#L22](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L22)

---
### [G-05] Comparison operators

---
**Description:**

In the EVM, there is no opcode for `>=` or `<=`. When using greater than or equal, two operations are performed: `>` and `=`. Using strict comparison operators hence saves gas.

**Recommendation:**

 Replace `<=` with `<`, and `>=` with `>`. Do not forget to increment/decrement the compared variable.

**Lines of Code:** 

- [Lottery.sol#L164](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L164)
- [Lottery.sol#L272](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L272)
- [Lottery.sol#L280](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L280)
- [LotterySetup.sol#L51](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L51)
- [LotterySetup.sol#L56](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L56)
- [LotterySetup.sol#L61](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L61)
- [LotterySetup.sol#L66](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L66)
- [LotterySetup.sol#L137](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L137)
- [RNSourceController.sol#L64](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L64)
- [ReferralSystem.sol#L62](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L62)
- [ReferralSystem.sol#L140](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L140)
- [TicketUtils.sol#L68](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L68)
- [TicketUtils.sol#L99](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L99)

---
### [G-06] Use a more recent version of solidity

---
**Description:**

Solidity 0.8.10 has a useful change that reduced gas costs of external calls which expect a return value.

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

**Lines of Code:** 

- [Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol)
- [LotteryMath.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryMath.sol)
- [LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol)
- [LotteryToken.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryToken.sol)
- [PercentageMath.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/PercentageMath.sol)
- [RNSourceBase.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceBase.sol)
- [RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol)
- [ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol)
- [Ticket.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol)
- [TicketUtils.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol)
- [VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol)
- [ILottery.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILottery.sol)
- [ILotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol)
- [ILotteryToken.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotteryToken.sol)
- [IRNSource.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IRNSource.sol)
- [IRNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IRNSourceController.sol)
- [IReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystem.sol)
- [IReferralSystemDynamic.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IReferralSystemDynamic.sol)
- [ITicket.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ITicket.sol)
- [IVRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IVRFv2RNSource.sol)
- [StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol)
- [Staking.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol)
- [IStakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/interfaces/IStakedTokenLock.sol)
- [IStaking.sol](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/interfaces/IStaking.sol)

---
### [G-07] `<array>.length` should not be looked up in every loop of a for-loop

---
**Description:**

The overheads outlined below are *PER LOOP*, excluding the first loop 

- storage arrays incur a Gwarmaccess (100 gas)

- memory arrays use `MLOAD` (3 gas)

- calldata arrays use `CALLDATALOAD` (3 gas)

Caching the length changes each of these to a `DUP<N>` (3 gas), and gets rid of the extra DUP<N> needed to store the stack offset



**Lines of Code:** 

- [Lottery.sol#L125](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L125)
- [ReferralSystem.sol#L35](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L35)
- [ReferralSystem.sol#L77](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L77)

---
### [G-08] `x += y` costs more gas than `x = x + y` for state variables

---
**Lines of Code:** 

- [Lottery.sol#L129](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L129)
- [Lottery.sol#L275](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L275)
- [ReferralSystem.sol#L64](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L64)
- [ReferralSystem.sol#L67](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L67)
- [ReferralSystem.sol#L69](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L69)
- [ReferralSystem.sol#L71](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L71)
- [StakedTokenLock.sol#L30](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L30)
- [StakedTokenLock.sol#L43](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L43)
---
### [G-09] Multiple `address` mappings can be combined into a single `mapping` of an `address` to a `struct`, where appropriate

---
**Description:**

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot

**Lines of Code:** 

- [Staking.sol#L19-L20](https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L19-L20)

