# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1  | Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct | 11 |
| 2  | Use `unchecked` blocks to save gas  |  1 |
| 3  | `storage` variable should be cached into `memory` variables instead of re-reading them  |  4 |
| 4  | Avoid struct variables default value initialization  |  1 |
| 5  | `memory` values should be emitted in events instead of `storage` ones  |  1 |
| 6  | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables  |  3 |


## Findings


### 1- Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct :

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

There are 4 instances of this issue in `Lottery` :

```solidity
File: Lottery.sol

27      mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
36      mapping(uint128 => uint120) public override winningTicket;
37      mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
39      mapping(uint128 => uint256) public override ticketsSold;
```

These mappings could be refactored into the following struct and mapping for example :

```solidity
struct Draw {
    uint120 winningTicket;
    uint256 ticketsSold;
    mapping(uint8 => uint256) winAmount;
    mapping(uint120 => uint256) unclaimedCount;
}
    
mapping(uint128 => Draw) public drawsMap;
```

There are 5 instances of this issue in `ReferralSystem` :

```solidity
File: ReferralSystem.sol

17      mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;
19      mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;
21      mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;
23      mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;
25      mapping(uint128 => uint256) public override minimumEligibleReferrals;
```

These mappings could be refactored into the following struct and mapping for example :

```solidity
struct Draw {
    uint256 totalTicketsForReferrersPerDraw;
    uint256 referrerRewardPerDrawForOneTicket;
    uint256 playerRewardsPerDrawForOneTicket;
    uint256 minimumEligibleReferrals;
    mapping(address => UnclaimedTicketsData) unclaimedTickets;
}
    
mapping(address => Draw) public drawsMap;
```

There are 2 instances of this issue in `Staking` :

```solidity
File: staking/Staking.sol

19      mapping(address => uint256) public override userRewardPerTokenPaid;
20      mapping(address => uint256) public override rewards;
```

These mappings could be refactored into the following struct and mapping for example :

```solidity
struct UserRewards {
    uint256 userRewardPerTokenPaid;
    uint256 rewards;
}
    
mapping(address => UserRewards) public userRewards;
```

### 2- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There is 1 instances of this issue:

File: Lottery.sol [Line 273](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L273)
```solidity
uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
```

In the code linked above the operation cannot underflow due to the checks that preceeds it [Line 272](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L272) so it should be marked as `unchecked`. 


### 3- `storage` variable should be cached into `memory` variables instead of re-reading them :

The instances below point to the second+ access of a state variable within a function, the caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read, thus saves **100gas** for each instance.

There are 4 instances of this issue :

File: Lottery.sol [Line 135-141](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L135-L141)

In the code linked above the value of `currentDraw` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

File: Lottery.sol [Line 272-273](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L272-L273)

In the code linked above the value of `currentDraw` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

File: ReferralSystem.sol [Line 62-65](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L62-L65)

In the code linked above the value of `unclaimedTickets[currentDraw][referrer].referrerTicketCount` is read multiple times (3) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

File: RNSourceBase.sol [Line 34-37](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L34-L37)

In the code linked above the value of `requests[requestId].status` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.


### 4- Avoid struct variables default value initialization :

Whan saving a new struct data to storage there is no need to initialize variables to their default values (0 for uint or int) as it will cost more gas, only the non default value should be stored.

There is 1 instances of this issue :

File: RNSourceBase.sol [Line 27](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L27)
```solidity
requests[requestId] = RandomnessRequest({ status: RequestStatus.Pending, randomNumber: 0 });
```

This should be rewritten as follow to save gas : 

```solidity
requests[requestId].status = RequestStatus.Pending;
```

As the `requests[requestId].randomNumber` value is already equal to 0 by default, it should not be set again.


### 5- `memory` values should be emitted in events instead of `storage` ones :

The values emitted in events shouldn’t be read from storage but the existing memory values should be used instead, this will save **~101 GAS**.

There is 1 instance of this issue :

File: RNSourceController.sol [Line 73](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L73)
```solidity
emit Retry(source, failedSequentialAttempts);
```

The memory value `failedAttempts` should be emitted as follow :

```solidity
emit Retry(source, failedAttempts);
```

### 6- `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables :

Using the addition operator instead of plus-equals saves **113 gas** as explained [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 3 instances of this issue:

```solidity
File: staking/StakedTokenLock.sol

30      depositedBalance += amount;
43      depositedBalance -= amount;

File: Lottery.sol

275     currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```