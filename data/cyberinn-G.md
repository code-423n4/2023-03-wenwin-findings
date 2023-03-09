# [G-01] internal functions only called once can be inlined to save gas

## Impact

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

## Findings

Total:1

[src/LotterySetup.sol#L160](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/LotterySetup.sol#L160)

```solidity
160:    function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {
```

# [G-02] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

## Impact

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

## Findings

Total:1

[src/staking/Staking.sol#L19-L20](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/staking/Staking.sol#L19-L20)

```solidity
19:    mapping(address => uint256) public override userRewardPerTokenPaid;
20:    mapping(address => uint256) public override rewards;
```

# [G-03] x += y or x -= y costs more gas than x = x + y or x = x - y for state variables

### Impact

Using the addition operator instead of plus-equals saves 113 gas. Usually does not work with struct and mappings.

### Findings

Total:6

[src/staking/StakedTokenLock.sol#L43](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/staking/StakedTokenLock.sol#L43)

```solidity
43:    depositedBalance -= amount;
```

[src/staking/StakedTokenLock.sol#L30](https://github.com/code-423n4/2023-03-wenwin/tree/main//src/staking/StakedTokenLock.sol#L30)

```solidity
30:    depositedBalance += amount;
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