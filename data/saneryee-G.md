# Gas Optimizations Report

|         | Issue                                                                                                                              | Instances |
| ------- | :--------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-001] | x += y or x -= y costs more gas than x = x + y or x = x - y for state variables                                                    |     7     |
| [G-002] | `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping` |     2     |
| [G-003] | Functions guaranteed to revert when called by normal users can be marked payable                                                   |     4     |
| [G-004] | Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate                 |     1     |
| [G-005] | internal functions only called once can be inlined to save gas                                                                     |     1     |
| [G-006] | Replace modifier with function                                                                                                     |     3     |
| [G-007] | Expressions that cannot be overflowed can be unchecked                                                                             |     2     |

## [G-001] x += y or x -= y costs more gas than x = x + y or x = x - y for state variables

### Impact

Using the addition operator instead of plus-equals saves 113 gas. Usually does not work with struct and mappings.

### Findings

Total:7

[src/staking/StakedTokenLock.sol#L43](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/staking/StakedTokenLock.sol#L43)

```solidity
43:    depositedBalance -= amount;
```

[src/staking/StakedTokenLock.sol#L30](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/staking/StakedTokenLock.sol#L30)

```solidity
30:    depositedBalance += amount;
```

[src/TicketUtils.sol#L96](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/TicketUtils.sol#L96)

```solidity
96:    winTier += uint8(intersection & uint120(1));
```

[src/TicketUtils.sol#L29](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/TicketUtils.sol#L29)

```solidity
29:    ticketSize += (ticket & uint256(1));
```

[src/LotteryMath.sol#L84](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/LotteryMath.sol#L84)

```solidity
84:    bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
```

[src/LotteryMath.sol#L55](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/LotteryMath.sol#L55)

```solidity
55:    newProfit -= int256(expectedRewardsOut);
```

[src/LotteryMath.sol#L64](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/LotteryMath.sol#L64)

```solidity
64:    excessPotInt -= int256(fixedJackpotSize);
```

## [G-002] `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping`

### Impact

It may not be obvious, but every time you copy a storage `struct/array/mapping` to a `memory` variable, you are literally copying each member by reading it from `storage`, which is expensive. And when you use the `storage` keyword, you are just storing a pointer to the storage, which is much cheaper.

### Findings

Total:2

[src/TicketUtils.sol#L54](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/TicketUtils.sol#L54)

```solidity
54:    uint8[] memory numbers = new uint8[](selectionSize);
```

[src/TicketUtils.sol#L63](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/TicketUtils.sol#L63)

```solidity
63:    bool[] memory selected = new bool[](selectionMax);
```

### Recommendation

Using `storage` instead of `memory`

## [G-003] Functions guaranteed to revert when called by normal users can be marked payable

### Impact

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings

Total:4

[src/staking/StakedTokenLock.sol#L24](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/staking/StakedTokenLock.sol#L24)

```solidity
24:    function deposit(uint256 amount) external override onlyOwner {
```

[src/staking/StakedTokenLock.sol#L37](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/staking/StakedTokenLock.sol#L37)

```solidity
37:    function withdraw(uint256 amount) external override onlyOwner {
```

[src/RNSourceController.sol#L77](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/RNSourceController.sol#L77)

```solidity
77:    function initSource(IRNSource rnSource) external override onlyOwner {
```

[src/RNSourceController.sol#L89](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/RNSourceController.sol#L89)

```solidity
89:    function swapSource(IRNSource newSource) external override onlyOwner {
```

### Recommendation

Mark the function as payable.

## [G-004] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Impact

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

### Findings

Total:1

[src/staking/Staking.sol#L19-L20](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/staking/Staking.sol#L19-L20)

```solidity
19:    mapping(address => uint256) public override userRewardPerTokenPaid;
20:        mapping(address => uint256) public override rewards;
```

## [G-005] internal functions only called once can be inlined to save gas

### Impact

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

### Findings

Total:1

[src/LotterySetup.sol#L160](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/LotterySetup.sol#L160)

```solidity
160:    function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {
```

## [G-006] Replace modifier with function

### Impact

Modifiers make code more elegant, but cost more than normal functions.

### Findings

Total:3

[src/LotterySetup.sol#L104](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/LotterySetup.sol#L104)

```solidity
104:    modifier requireJackpotInitialized() {
```

[src/Lottery.sol#L60](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/Lottery.sol#L60)

```solidity
60:    modifier onlyWhenExecutingDraw() {
```

[src/Lottery.sol#L52](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/Lottery.sol#L52)

```solidity
52:    modifier whenNotExecutingDraw() {
```

## [G-007] Expressions that cannot be overflowed can be unchecked

### Impact

There are several cases that do not lead to overflow and can be unchecked.

- a previous push in array
- If an underflow occurs, the next statement will revert
- There is a check in the previous code.

### Findings

Total:2

[src/LotterySetup.sol#L171](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/LotterySetup.sol#L171)

```solidity
171:    if ((rewards[winTier] % divisor) != 0) {
```

[src/TicketUtils.sol#L58](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/TicketUtils.sol#L58)

```solidity
58:    numbers[i] = uint8(randomNumber % currentSelectionCount);
```
