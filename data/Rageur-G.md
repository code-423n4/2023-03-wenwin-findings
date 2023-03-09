## GAS-1: <X> += <Y> costs more gas than <X> = <X> + <Y> for state variables

### Description

Using the addition operator instead of plus-equals saves gas.

### Affected file

* Lottery.sol (Line: 129, 275)
* StakedTokenLock.sol (Line: 30, 43)

## GAS-2: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* LotterySetup.sol (Line: 51)

## GAS-3: Functions guaranteed to revert when called by normal users can be marked payable

### Description

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

### Affected file

* Lottery.sol (Line: 203, 259)
* StakedTokenLock.sol (Line: 24, 37)

## GAS-4: Internal functions not called by the contract should be removed to save deployment gas

### Description

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

### Affected file

* Lottery.sol (Line: 203, 279, 285)
* Staking.sol (Line: 108)
* VRFv2RNSource.sol (Line: 28, 32)

## GAS-5: Internal functions only called once can be inlined to save gas

### Description

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

### Affected file

* LotterySetup.sol (Line: 160)

## GAS-6: Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Description

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

### Affected file

* Staking.sol (Line: 19, 20)

## GAS-7: Not using the named return variables when a function returns, wastes deployment gas

### Description

It is not necessary to have both a named return and a return statement.

### Affected file

* Lottery.sol (Line: 234, 238)
* LotterySetup.sol (Line: 120, 120, 120)
* Staking.sol (Line: 48, 48, 61)

## GAS-8: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* Lottery.sol (Line: 234)
* LotterySetup.sol (Line: 120)

## GAS-9: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* Lottery.sol (Line: 44, 52, 60, 69)
* LotterySetup.sol (Line: 104, 112)

## GAS-10: Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

### Affected file

* Lottery.sol (Line: 162, 204, 205, 211, 234, 238, 273, 279)
* LotterySetup.sol (Line: 112, 120, 152, 156, 169, 170)
* VRFv2RNSource.sol (Line: 17, 18)

## GAS-11: Use require() instead of assert()

### Description

The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

### Affected file

* LotterySetup.sol (Line: 147)

## GAS-12: Using Solidity version 0.8.17 will provide an overall gas optimization

### Description

Using at least 0.8.10 will save gas due to skipped extcodesize check if there is a return value.

### Affected file

* VRFv2RNSource.sol (Line: 3)