
- [Gas](#gas)
    - [**1. Reorder structure layout**](#1-reorder-structure-layout)
    - [**2. Optimize conditional**](#2-optimize-conditional)
    - [**3. Use require instead of assert**](#3-use-require-instead-of-assert)
    - [**4. Use local variable instead of storage**](#4-use-local-variable-instead-of-storage)


# Gas

## **1. Reorder structure layout**

The following structures could be optimized moving the position of certain values in order to save some storage slots:

```diff
struct LotterySetupParams {
+   /// @dev Count of numbers user picks for the ticket
+   uint8 selectionSize;
+   /// @dev Max number user can pick
+   uint8 selectionMax;
    /// @dev Token to be used as reward token for the lottery
    IERC20 token;
    /// @dev Parameters of the draw schedule for the lottery
    LotteryDrawSchedule drawSchedule;
    /// @dev Price to pay for playing single game (including fee)
    uint256 ticketPrice;
-   /// @dev Count of numbers user picks for the ticket
-   uint8 selectionSize;
-   /// @dev Max number user can pick
-   uint8 selectionMax;
    /// @dev Expected payout for one ticket, expressed in `rewardToken`
    uint256 expectedPayout;
    /// @dev Array of fixed rewards per each non jackpot win
    uint256[] fixedRewards;
}
```

**Reference:**

- https://ethdebug.github.io/solidity-data-representation

> Enums are represented by integers; the possibility listed first by 0, the next by 1, and so forth. An enum type just acts like uintN, where N is the smallest legal value large enough to accomodate all the possibilities.

- https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#storage-inplace-encoding

**Affected source code:**

- [ILotterySetup.sol:71-73](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/ILotterySetup.sol#L71-L73)
- [ILotterySetup.sol:94](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/ILotterySetup.sol#L94)

## **2. Optimize conditional**

```diff
-       bool notEnoughRetryInvocations = failedSequentialAttempts < maxFailedAttempts;
-       bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;
-       if (notEnoughRetryInvocations || notEnoughTimeReachingMaxFailedAttempts) {
+       if ((failedSequentialAttempts < maxFailedAttempts)) || (block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay)) {
            revert NotEnoughFailedAttempts();
        }
```

There is an OR conditional that previously computes both conditions. The advantage of an OR conditional is that if one condition is met, the code of the rest of the conditional will not be executed, and therefore gas will be saved, so by modifying the code as shown previously, an optimization will be made when reverting said method.

**Affected source code:**

- [RNSourceController.sol:93-97](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L93-L97)

**Some gas differences:**

```diff
- [PASS] testSuccessfulSwapSource() (gas: 101837)
+ [PASS] testSuccessfulSwapSource() (gas: 101816)
- [PASS] testSwapSourceFailsWithInsufficientFailedAttempts() (gas: 85632)
+ [PASS] testSwapSourceFailsWithInsufficientFailedAttempts() (gas: 83427)
- [PASS] testSwapSourceFailsWithInsufficientFailedAttemptsWhenNotEnoughTimeSinceReachingMaxFailedAttempts() (gas: 114248)
+ [PASS] testSwapSourceFailsWithInsufficientFailedAttemptsWhenNotEnoughTimeSinceReachingMaxFailedAttempts() (gas: 114226)
```

## **3. Use `require` instead of `assert`**

The `assert()` and `require()` functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the `assert()` function when false, **uses up all the remaining gas and reverts all the changes made.**

Meanwhile, a `require()` function when false, also reverts back all the changes made to the contract but **does refund all the remaining gas fees we offered to pay**. This is the most common Solidity function used by developers for debugging and error handling.

**Affected source code:**

- [LotterySetup.sol:147](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L147)

**Some gas differences:**

```diff
- [PASS] testFinalizeInitialPot(uint256,uint256) (runs: 256, μ: 8541365, ~: 8536222)
+ [PASS] testFinalizeInitialPot(uint256,uint256) (runs: 256, μ: 8540713, ~: 8535622)
```

## **4. Use local variable instead of storage**

Since the same value is previously stored in a local variable, it is cheaper to query its contents than to access the storage for such a query, so the following modification is recommended:

```diff
    function finalizeInitialPotRaise() external override {
        if (initialPot > 0) {
            revert JackpotAlreadyInitialized();
        }
        // slither-disable-next-line timestamp
        if (block.timestamp <= initialPotDeadline) {
            revert FinalizingInitialPotBeforeDeadline();
        }
        uint256 raised = rewardToken.balanceOf(address(this));
        if (raised < minInitialPot) {
            revert RaisedInsufficientFunds(raised);
        }
        initialPot = raised;

        // must hold after this call, this will be used as a check that jackpot is initialized
-       assert(initialPot > 0);
+       assert(raised > 0);

        emit InitialPotPeriodFinalized(raised);
    }
```

**Affected source code:**

- [LotterySetup.sol:147](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L147)

**Some gas differences:**

```diff
- [PASS] testFinalizeInitialPot(uint256,uint256) (runs: 256, μ: 8541365, ~: 8536222)
+ [PASS] testFinalizeInitialPot(uint256,uint256) (runs: 256, μ: 8541168, ~: 8536222)
```
