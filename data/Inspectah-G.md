# WenWin

# Gas Optimizations

## 1. Avoiding the use of modifiers when storage variable reads are required.

```solidity
contract TP{
    error DrawAlreadyInProgress();
    bool public drawExecutionInProgress;
    uint256 testVar;

    modifier whenNotExecutingDraw() {
        if (drawExecutionInProgress) {
            revert DrawAlreadyInProgress();
        }
        _;
    }
    function newFunc() public whenNotExecutingDraw {
        testVar+=1;
    }
    function newFunc2() public {
        whenNotExecutingDrawFunc();
        testVar+=1;
    }
    function whenNotExecutingDrawFunc() public view  {
        if(drawExecutionInProgress){
            revert DrawAlreadyInProgress();
        }
    }
}
```

`newFunc` and `newFunc2` both do the thing but `newFunc`uses a modifer to check for state variable `drawExecutionInProgress` which makes it more expensive (45654 gas) than its alternative `newFunc2` which uses a helper function `whenNotExecutingDrawFunc` to make it same checks (28644 gas).

### Instances:

`Lottery.sol` : [L-60](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L60), [L-52](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L52)

`LotterySetup.sol` : [L-104](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L104) 

## 2. Compound assignment operators `+=` and `-=` should be replaced with the normal syntax.

```solidity
function claimWinningTickets(uint256[] calldata ticketIds) external  returns  (uint256 claimedAmount) {
        uint256 totalTickets = ticketIds.length;
        for (uint256 i = 0; i < totalTickets; ++i) {
            claimedAmount += claimWinningTicket(ticketIds[i]);
        }
        testVar=testVar+1;
        // rewardToken.safeTransfer(msg.sender, claimedAmount);
    }

    function claimWinningTickets2(uint256[] calldata ticketIds) external  returns  (uint256 claimedAmount) {
        uint256 totalTickets = ticketIds.length;
        for (uint256 i = 0; i < totalTickets; ++i) {
            claimedAmount = claimedAmount+claimWinningTicket(ticketIds[i]);
        }
        testVar=testVar+1;
        // rewardToken.safeTransfer(msg.sender, claimedAmount);
    }
    function claimWinningTicket(uint256) private pure returns(uint){
        //functionLogic
    }
```

`claimWinningTickets` uses compound assignment operator `+=` whereas `claimWinningTickets2` uses the normal syntax. It results in massive difference in the transaction cost in terms of gas. The former function consumes  48037 gas whereas the latter cnsumes 30849 gas when the `ticketIds.length` is 5.

### Instances:

`Lottery.sol` : [L-129](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L129), [L-173](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L173), [L-275](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L275)

`LotteryMath.sol` : [L-84](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L84),

`ReferralSystem.sol` : [L-64](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L64), [L-67](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L67), [L-69](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L69), [L-71](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L71), [L-78](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L78), [L-147](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L147)

`TicketUtils.sol`: [L-29](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L29), [L-96](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L96)

`staking/StakedTokenLock.sol` : [L-30](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L30)

## 3. Using `require()` instead of `assert()`

This is recommended because if the latter fails, all the remaining gas is used up before reverting the changes made whereas `require()` returns back the remaining gas.

### Instances:

`LotterySetup.sol` : [L-147](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L147)

`TicketUtils.sol`: [L-99](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L99)

## 4.Use `calldata` instead of `memory` for external functions where the function argument is read-only.

In some cases, having function arguments in calldata instead ofmemory is more optimal.

Consider the following generic example:

 

```solidity
contract C {
function add(uint[] memory arr) external returns (uint sum) {
  uint length = arr.length;
  for (uint i = 0; i < arr.length; i++) {
      sum += arr[i];
  }
}
}
```

In the above example, the dynamic array arr has the storage locationmemory. When the function gets called externally, the array values arekept in calldata and copied to memory during ABI decoding (using theopcode calldataload and mstore). And during the for loop, arr[i]accesses the value in memory using a mload. However, for the aboveexample this is inefficient. Consider the following snippet instead:

```solidity
contract C {
function add(uint[] calldata arr) external returns (uint sum) {
  uint length = arr.length;
  for (uint i = 0; i < arr.length; i++) {
      sum += arr[i];
  }
}
}
```

In the above snippet, instead of going via memory, the value is directlyread from calldata using calldataload. That is, there are nointermediate memory operations that carries this value.

Gas savings: In the former example, the ABI decoding begins withcopying value from calldata to memory in a for loop. Each iterationwould cost at least 60 gas. In the latter example, this can becompletely avoided. This will also reduce the number of instructions andtherefore reduces the deploy time cost of the contract.

In short, use calldata instead of memory if the function argumentis only read.

### Instances:

`ReferralSystem.sol` : [L-76](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L76)

## 5. **Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate.**

If both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

In `Staking.sol` mappings `userRewardPerTokenPaid` and `rewards` map addresses to uint256 type values. These mappings are then read together in functions `earned` and `_updateReward`

These 2 mappings can be combined into one address to struct mapping like:

```solidity
struct rewardVars{
	uint256 userRewardPerTokenPaid;
	uint256 rewards;
}
mapping(address=>rewardVars) combinedMapping;
```

## 6. ****Using unchecked blocks to save gas****

**Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block [see resource](https://github.com/ethereum/solidity/issues/10695).**

```solidity

if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {
            uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
}
```

The operation `currentDraw - LotteryMath.DRAWS_PER_YEAR` will never underflow because of the check on the previous line so it is safe to place it in a unchecked block to save gas.

## 7. If statements which use the `&&` operators can be split into 2 nested if statements to save gas.

### Instances:

`LotteryMath.sol` : [L-83](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L83)

`TicketUtils.sol` : [L-32](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L32)

`StakedTokenLock.sol`: [L-39](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L39)

The same concept works with revert/require statements:

 `TicketUtils.sol`: [L-99](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L99)

## 8. **Use assembly to check for address(0)**

Saves 6 gas per instance if using assembly to check for address(0)

e.g.

```solidity
assembly { if iszero(_addr) {  mstore(0x00, "zero address")  revert(0x00, 0x20) }}
```

Instances: 

`LotterySetup.sol` : [L-42](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L42)

`ReferralSystem.sol` :[L-60](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L60)

`RNSourceController.sol` :[L-78](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L78), [L-81](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L81), [L-90](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L90)

`Staking.sol` :[L-31](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L31), [L-34](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L34), [L-37](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L37), [L-109](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L109), [L-113](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L113)