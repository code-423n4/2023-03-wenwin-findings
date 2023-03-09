## FINDINGS
### [G-01] Using `delete` instead of setting state variable/mapping to `0` saves gas

8 results - 4 file:

```solidity
File: /staking/Staking.sol
	95: rewards[msg.sender] = 0;
```

- [Staking.sol#L95](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L95)

```solidity
File: /src/Lottery.sol
	255: frontendDueTicketSales[beneficiary] = 0;
```

- [Lottery.sol#L255](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L255)

```solidity
File: /src/RNSourceController.sol	
	52: failedSequentialAttempts = 0;
	53: maxFailedAttemptsReachedAt = 0;
	99: failedSequentialAttempts = 0;
	100: maxFailedAttemptsReachedAt = 0;
```

- [RNSourceController.sol#L52](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L52)

```solidity
File: /src/ReferralSystem.sol	
	142: unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
	148: unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;     
```

- [ReferralSystem.sol#L142](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L142)

### Mitigation

```jsx
File: /staking/Staking.sol
	- 95: rewards[msg.sender] = 0;
	+ 95: delete rewards[msg.sender];
```

### [G-02] ****x += y costs more gas than x = x + y for state variables****

```solidity
File: /src/staking/StakedTokenLock.sol
	30: depositedBalance += amount;
```

- [StakedTokenLock.sol#L30](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30)

### [G-03]Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas

When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. Each iteration of this for-loop costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Structs have the same overhead as an array of length one

```solidity
File: /src/ReferralSystem.sol
76 function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {

```

- [ReferralSystem.sol#L76](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L76)

### [G-04] Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. **Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

3 results - 2 files:

### [1] `RNSourceBase.sol.fulfill() : requests[requestId].status` should be cached in local storage

**See `@audit` tag:**

```solidity
File: /[src/RNSourceBase.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L33) 
33: function fulfill(uint256 requestId, uint256 randomNumber) internal {
34:     if (requests[requestId].status == RequestStatus.None) { //@audit: Initial access
35:         revert RequestNotFound(requestId);
36:     }
37:     if (requests[requestId].status == RequestStatus.Fulfilled) { //@audit:  2nd access
38:         revert RequestAlreadyFulfilled(requestId);
39:     }
40:
41:     requests[requestId].randomNumber = randomNumber;
42:     requests[requestId].status = RequestStatus.Fulfilled; //@audit: 3rd access
43:     IRNSourceConsumer(authorizedConsumer).onRandomNumberFulfilled(randomNumber);
44:
45:     emit RequestFulfilled(requestId, randomNumber);
46: }

```

- [RNSourceBase.sol#L33](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L33)

### [2] `ReferralSystem.sol.referralRegisterTickets() : unclaimedTickets[currentDraw][referrer]` should be cached in local storage

**See `@audit` tag:**

```solidity
File: /src/ReferralSystem.sol
	52: function referralRegisterTickets(

	60:     if (referrer != address(0)) {
	61:         uint256 minimumEligible = minimumEligibleReferrals[currentDraw];
	62:         if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) { //@audit: Initial access
	63:             if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) { //@audit: 2nd access
	64:                 totalTicketsForReferrersPerDraw[currentDraw] +=
	65:                     unclaimedTickets[currentDraw][referrer].referrerTicketCount; //@audit: 3rd access
	66:             }
	67:             totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
	68:         }
	69:         unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets); //@audit: 4th access
	70:     }
	71:     unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
	72: }

```

- [ReferralSystem.sol#L52](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L52)

### [3]`ReferralSystem.sol.referralRegisterTickets():totalTicketsForReferrersPerDraw[currentDraw]` should be cached in local storage

Instead of reading the variable twice, it can be read once and stored in a local variable to be used in the calculation.

**See `@audit` tag:**

```solidity
File:  /src/ReferralSystem.sol
52: function referralRegisterTickets(

60     if (referrer != address(0)) {
61         uint256 minimumEligible = minimumEligibleReferrals[currentDraw];
62         if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
63             if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
64                 totalTicketsForReferrersPerDraw[currentDraw] +=  //@audit: Initial access
65                     unclaimedTickets[currentDraw][referrer].referrerTicketCount;
66             }
67             totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets; //@audit: 2nd access
68         }
69         unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

```

- [ReferralSystem.sol#L52](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L52)

### [G-05] **Pre-increments and pre-decrements are cheaper than post-increments and post-decrements**

```solidity
File: /src/Lottery.sol#
	266: unclaimedCount[ticketsInfo[ticketId].drawId][ticketsInfo[ticketId].combination]--;
```

- [Lottery.sol#L266](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L266)

### [G-06] Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block[see resource](https://github.com/ethereum/solidity/issues/10695)

**See `@audit` tag:**

```solidity
File: /src/Lottery.sol
273:   uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR; //@audit gas: should be unchecked (can't underflow due to L272)
```

- [Lottery.sol#L273](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L273)

The operation `uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;` cannot underflow due to the check on [Line 272](https://github.com/code-423n4/2022-10-paladin/blob/d6d0c0e57ad80f15e9691086c9c7270d4ccfe0e6/contracts/WardenPledge.sol#L267) that ensures that `currentDraw` is greater than `LotteryMath.DRAWS_PER_YEAR` before perfoming the arithmetic operation

### [G-07] Cache array length outside of loop

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

**See `@audit` tag:**

```solidity
File: /src/VRFv2RNSource.sol
	32: function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {
	33:       if (randomWords.length != 1) { //@audit gas: array length should be cached
	34:            revert WrongRandomNumberCountReceived(requestId, randomWords.length); //@audit gas: array length should be cached
	35:        }
	36:
	37:        fulfill(requestId, randomWords[0]);
	38:    }
```

- [VRFv2RNSource.sol#L32](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L32)