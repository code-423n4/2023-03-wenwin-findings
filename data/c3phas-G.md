### Table of contents
- [FINDINGS](#findings)
  - [Note on Gas estimates.](#note-on-gas-estimates)
- [Tighly pack storage variables/optimize the order of variable declaration(Saves 2K gas in total)](#tighly-pack-storage-variablesoptimize-the-order-of-variable-declarationsaves-2k-gas-in-total)
  - [RNSourceController.sol: pack source together with lastRequestFulfilled(Saves 1 SLOT: 2K gas)](#rnsourcecontrollersol-pack-source-together-with-lastrequestfulfilledsaves-1-slot-2k-gas)
- [Pack structs by putting data types that fit together next to each other(Saves 2K gas in total)](#pack-structs-by-putting-data-types-that-fit-together-next-to-each-othersaves-2k-gas-in-total)
  - [ILotterySetup.sol: LotterySetupParams{}: Pack  uint8 selectionSize and  uint8 selectionMax with   IERC20 token(Saves 1 SLOT : 2k Gas)](#ilotterysetupsol-lotterysetupparams-pack--uint8-selectionsize-and--uint8-selectionmax-with---ierc20-tokensaves-1-slot--2k-gas)
- [Emitting storage values instead of the memory one.](#emitting-storage-values-instead-of-the-memory-one)
  - [RNSourceController.sol.retry(): Emit local variable instead of storage (Saves ~100 gas)](#rnsourcecontrollersolretry-emit-local-variable-instead-of-storage-saves-100-gas)
- [A modifier used only once and not being inherited should be inlined to save gas](#a-modifier-used-only-once-and-not-being-inherited-should-be-inlined-to-save-gas)
- [Use the cached value Instead of fetching the storage value which is expensive](#use-the-cached-value-instead-of-fetching-the-storage-value-which-is-expensive)
  - [LotterySetup.sol.finalizeInitialPotRaise(): Use the cached value - saves ~100 gas](#lotterysetupsolfinalizeinitialpotraise-use-the-cached-value---saves-100-gas)
- [Multiple accesses of a mapping/array should use a local variable cache](#multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache)
  - [RNSourceBase.sol.fulfill(): requests\[requestId\] should be cached in local storage](#rnsourcebasesolfulfill-requestsrequestid-should-be-cached-in-local-storage)
  - [ReferralSystem.sol.referralRegisterTickets(): unclaimedTickets\[currentDraw\]\[referrer\] should be cached in local storage](#referralsystemsolreferralregistertickets-unclaimedticketscurrentdrawreferrer-should-be-cached-in-local-storage)
  - [ReferralSystem.sol.referralRegisterTickets(): totalTicketsForReferrersPerDraw\[currentDraw\] should be cached in local storage](#referralsystemsolreferralregistertickets-totalticketsforreferrersperdrawcurrentdraw-should-be-cached-in-local-storage)
- [x += y costs more gas than x = x + y for state variables](#x--y-costs-more-gas-than-x--x--y-for-state-variables)
- [Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead](#usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead)
- [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)

## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Through out the report some places might be denoted with audit tags to show the actual place affected.

### Note on Gas estimates.
I've tried to give the exact amount of gas being saved from running the included tests. Whenever the function is within the test coverage, the average gas before and after will be included, and often a diff of the code will also accompany this.

Some functions are not covered by the test cases or are internal/private functions. In this case, the gas can be estimated by looking at the opcodes involved. Whenever this method is applied the gas saved is stated as an approximate ie `~` symbol will be used

## Tighly pack storage variables/optimize the order of variable declaration(Saves 2K gas in total)

Here, the storage variables can be tightly packed 

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L11-L16
### RNSourceController.sol: pack source together with lastRequestFulfilled(Saves 1 SLOT: 2K gas)
```solidity
File: /src/RNSourceController.sol
11:    IRNSource public override source;

13:    uint256 public override failedSequentialAttempts;
14:    uint256 public override maxFailedAttemptsReachedAt;
15:    uint256 public override lastRequestTimestamp;
16:    bool public override lastRequestFulfilled = true;
```

```diff
diff --git a/src/RNSourceController.sol b/src/RNSourceController.sol
index 2bd8cfc..26057f1 100644
--- a/src/RNSourceController.sol
+++ b/src/RNSourceController.sol
@@ -9,11 +9,11 @@ import "src/interfaces/IRNSourceController.sol";
 /// @dev A contract that controls the list of random number sources and dispatches random number requests to them.
 abstract contract RNSourceController is Ownable2Step, IRNSourceController {
     IRNSource public override source;
+    bool public override lastRequestFulfilled = true;

     uint256 public override failedSequentialAttempts;
     uint256 public override maxFailedAttemptsReachedAt;
     uint256 public override lastRequestTimestamp;
-    bool public override lastRequestFulfilled = true;
     uint256 public immutable override maxFailedAttempts;
     uint256 public immutable override maxRequestDelay;
     uint256 private constant MAX_MAX_FAILED_ATTEMPTS = 10;
```

## Pack structs by putting data types that fit together next to each other(Saves 2K gas in total)

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage ~20000 gas

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/ILotterySetup.sol#L63C28-L78
### ILotterySetup.sol: LotterySetupParams{}: Pack  uint8 selectionSize and  uint8 selectionMax with   IERC20 token(Saves 1 SLOT : 2k Gas)
```solidity
File: /src/interfaces/ILotterySetup.sol
63: struct LotterySetupParams {
64:    /// @dev Token to be used as reward token for the lottery
65:    IERC20 token;
66:    /// @dev Parameters of the draw schedule for the lottery
67:    LotteryDrawSchedule drawSchedule;
68:    /// @dev Price to pay for playing single game (including fee)
69:    uint256 ticketPrice;
70:    /// @dev Count of numbers user picks for the ticket
71:    uint8 selectionSize;
72:    /// @dev Max number user can pick
73:    uint8 selectionMax;
74:    /// @dev Expected payout for one ticket, expressed in `rewardToken`
75:    uint256 expectedPayout;
76:    /// @dev Array of fixed rewards per each non jackpot win
77:    uint256[] fixedRewards;
78: }
```

```diff
diff --git a/src/interfaces/ILotterySetup.sol b/src/interfaces/ILotterySetup.sol
index 780d1a3..b0642b6 100644
--- a/src/interfaces/ILotterySetup.sol
+++ b/src/interfaces/ILotterySetup.sol
@@ -61,16 +61,16 @@ struct LotteryDrawSchedule {

 /// @dev Parameters used to setup a new lottery
 struct LotterySetupParams {
+    /// @dev Count of numbers user picks for the ticket
+    uint8 selectionSize;
+    /// @dev Max number user can pick
+    uint8 selectionMax;
     /// @dev Token to be used as reward token for the lottery
     IERC20 token;
     /// @dev Parameters of the draw schedule for the lottery
     LotteryDrawSchedule drawSchedule;
     /// @dev Price to pay for playing single game (including fee)
     uint256 ticketPrice;
-    /// @dev Count of numbers user picks for the ticket
-    uint8 selectionSize;
-    /// @dev Max number user can pick
-    uint8 selectionMax;
     /// @dev Expected payout for one ticket, expressed in `rewardToken`
     uint256 expectedPayout;
     /// @dev Array of fixed rewards per each non jackpot win
```

## Emitting storage values instead of the memory one.
Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L60-L75
### RNSourceController.sol.retry(): Emit local variable instead of storage (Saves ~100 gas)
```solidity
File: /src/RNSourceController.sol
60:    function retry() external override {

69:        uint256 failedAttempts = ++failedSequentialAttempts;
70:        if (failedAttempts == maxFailedAttempts) {
71:            maxFailedAttemptsReachedAt = block.timestamp;
72:        }

73:        emit Retry(source, failedSequentialAttempts);
74:        requestRandomNumberFromSource();
75:    }
```

From running tests emitting `failedSequentialAttempts` and `failedAttempts` both return the same thing. As `failedAttempts` is a local variable we could save around 100 gas
```diff
diff --git a/src/RNSourceController.sol b/src/RNSourceController.sol
index 2bd8cfc..28f317b 100644
--- a/src/RNSourceController.sol
+++ b/src/RNSourceController.sol
@@ -70,7 +70,7 @@ abstract contract RNSourceController is Ownable2Step, IRNSourceController {
             maxFailedAttemptsReachedAt = block.timestamp;
         }

-        emit Retry(source, failedSequentialAttempts);
+        emit Retry(source, failedAttempts);
         requestRandomNumberFromSource();
     }
```


## A modifier used only once and not being inherited should be inlined to save gas
**Interesting optimizations explained, note the explanations**
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L104-L110
```solidity
File: /src/LotterySetup.sol
104:    modifier requireJackpotInitialized() {
105:        // slither-disable-next-line incorrect-equality
106:        if (initialPot == 0) {
107:            revert JackpotNotInitialized();
108:        }
109:        _;
110:    }
```

The above modifier is only called once via inheritance in the following

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L110-L120
```solidity
File: /src/Lottery.sol
110:    function buyTickets(
111:        uint128[] calldata drawIds,
112:        uint120[] calldata tickets,
113:        address frontend,
114:        address referrer
115:    )
116:        external
117:        override
118:        requireJackpotInitialized
119:        returns (uint256[] memory ticketIds)
120:    {
```


We can modify the function above as
```diff
diff --git a/src/Lottery.sol b/src/Lottery.sol
index 28d3477..2e21615 100644
--- a/src/Lottery.sol
+++ b/src/Lottery.sol
@@ -115,9 +115,11 @@ contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceCont
     )
         external
         override
-        requireJackpotInitialized
         returns (uint256[] memory ticketIds)
     {
+        if (initialPot == 0) {
+            revert JackpotNotInitialized();
+        }
         if (drawIds.length != tickets.length) {
             revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
         }
```

We might go fully optimizor mode here and optimize the order of the IF's statement to have the cheaper checks first as the check for `initialPot` involves reading a  storage variable vis-a-vis the second if statement that compares two local variables.
If we revert on the second check as it is, we would waste a lot of gas reading from storage.
**IF's/require() statements that check input arguments should be at the top of the function**

```diff
diff --git a/src/Lottery.sol b/src/Lottery.sol
index 28d3477..c7dff37 100644
--- a/src/Lottery.sol
+++ b/src/Lottery.sol
@@ -115,12 +115,14 @@ contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceCont
     )
         external
         override
-        requireJackpotInitialized
         returns (uint256[] memory ticketIds)
     {
         if (drawIds.length != tickets.length) {
             revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
         }
+        if (initialPot == 0) {
+            revert JackpotNotInitialized();
+        }
         ticketIds = new uint256[](tickets.length);
         for (uint256 i = 0; i < drawIds.length; ++i) {
             ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
```

**Other instances**
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L112-L118
```solidity
File: /src/LotterySetup.sol
112:    modifier beforeTicketRegistrationDeadline(uint128 drawId) {
113:        // slither-disable-next-line timestamp
114:        if (block.timestamp > ticketRegistrationDeadline(drawId)) {
115:            revert TicketRegistrationClosed(drawId);
116:        }
117:        _;
118:    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L44
```solidity
File: /src/Lottery.sol
44:    modifier requireValidTicket(uint256 ticket) {

52:    modifier whenNotExecutingDraw() {

60:    modifier onlyWhenExecutingDraw() {

69:    modifier onlyTicketOwner(uint256 ticketId) {
```

## Use the cached value Instead of fetching the storage value which is expensive

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L132-L150
### LotterySetup.sol.finalizeInitialPotRaise(): Use the cached value - saves ~100 gas
```solidity
File: /src/LotterySetup.sol
132:    function finalizeInitialPotRaise() external override {

144:        initialPot = raised;


146:        // must hold after this call, this will be used as a check that jackpot is initialized
147:        assert(initialPot > 0);

149:        emit InitialPotPeriodFinalized(raised);
150:    }
```

On  [Line 144](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L144) `initialPot` is set equal to `raised` thus rather than read `initialPot` which is a storage variable, after that asignment we should read it's local variable equivalent ie `raised`

```diff
diff --git a/src/LotterySetup.sol b/src/LotterySetup.sol
index 9c70086..1dfd9e0 100644
--- a/src/LotterySetup.sol
+++ b/src/LotterySetup.sol
@@ -98,7 +98,7 @@ contract LotterySetup is ILotterySetup {
             lotterySetupParams.selectionMax,
             lotterySetupParams.expectedPayout,
             lotterySetupParams.fixedRewards
-        );
+            );
     }

     modifier requireJackpotInitialized() {
@@ -144,7 +144,7 @@ contract LotterySetup is ILotterySetup {
         initialPot = raised;

         // must hold after this call, this will be used as a check that jackpot is initialized
-        assert(initialPot > 0);
+        assert(raised > 0);

         emit InitialPotPeriodFinalized(raised);
     }
```

## Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time.
**Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. 
As an example, instead of repeatedly calling ```someMap[someIndex]```, save its reference like this: ```SomeStruct storage someStruct = someMap[someIndex]``` and use it.

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceBase.sol#L33-L46
### RNSourceBase.sol.fulfill(): requests\[requestId] should be cached in local storage
```solidity
File: /src/RNSourceBase.sol
33:    function fulfill(uint256 requestId, uint256 randomNumber) internal {
34:        if (requests[requestId].status == RequestStatus.None) {//@audit: 1st access
35:            revert RequestNotFound(requestId);
36:        }
37:        if (requests[requestId].status == RequestStatus.Fulfilled) {//@audit: 2nd access
38:            revert RequestAlreadyFulfilled(requestId);
39:        }

41:        requests[requestId].randomNumber = randomNumber;//@audit: 3rd access
42:        requests[requestId].status = RequestStatus.Fulfilled; //@audit: 4th access
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L52-L72
### ReferralSystem.sol.referralRegisterTickets(): unclaimedTickets\[currentDraw]\[referrer] should be cached in local storage
```solidity
File: /src/ReferralSystem.sol
52:    function referralRegisterTickets(

60:        if (referrer != address(0)) {
61:            uint256 minimumEligible = minimumEligibleReferrals[currentDraw];
62:            if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
63:                if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
64:                    totalTicketsForReferrersPerDraw[currentDraw] +=
65:                        unclaimedTickets[currentDraw][referrer].referrerTicketCount;
66:                }

69:            unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
70:        }
71:        unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
72:    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L52-L72
### ReferralSystem.sol.referralRegisterTickets(): totalTicketsForReferrersPerDraw[currentDraw] should be cached in local storage
```solidity
File: /src/ReferralSystem.sol
52:    function referralRegisterTickets(

63:                if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
64:                    totalTicketsForReferrersPerDraw[currentDraw] +=
65:                        unclaimedTickets[currentDraw][referrer].referrerTicketCount;
66:                }
```



## x += y costs more gas than x = x + y for state variables

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L30
**Save 10 gas on average**
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2518    | 33452   | 42286 | 42286 |
| After  | 2518    | 33442   | 42273 | 42273 |

```solidity
File: /src/staking/StakedTokenLock.sol
30:        depositedBalance += amount;
```

```diff
diff --git a/src/staking/StakedTokenLock.sol b/src/staking/StakedTokenLock.sol
index f756d4d..8c965bc 100644
--- a/src/staking/StakedTokenLock.sol
+++ b/src/staking/StakedTokenLock.sol
@@ -27,7 +27,7 @@ contract StakedTokenLock is IStakedTokenLock, Ownable2Step {
             revert DepositPeriodOver();
         }

-        depositedBalance += amount;
+        depositedBalance = depositedBalance + amount;

```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L43
**Saves 9 gas per average**
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 576    | 13443   | 19377 | 19443 |
| After  | 576    | 13434   | 19364 | 19430 |

```solidity
File: /src/staking/StakedTokenLock.sol
43:        depositedBalance -= amount;
```

```diff
diff --git a/src/staking/StakedTokenLock.sol b/src/staking/StakedTokenLock.sol
index f756d4d..8c09989 100644
--- a/src/staking/StakedTokenLock.sol
+++ b/src/staking/StakedTokenLock.sol
@@ -40,7 +40,7 @@ contract StakedTokenLock is IStakedTokenLock, Ownable2Step {
             revert LockPeriodOngoing();
         }

-        depositedBalance -= amount;
+        depositedBalance = depositedBalance - amount;

```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L275
```solidity
File: /src/Lottery.sol
275:            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

```diff
diff --git a/src/Lottery.sol b/src/Lottery.sol
index 28d3477..d1adb28 100644
--- a/src/Lottery.sol
+++ b/src/Lottery.sol
@@ -272,7 +272,7 @@ contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceCont
         if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {
             uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
             uint256 unclaimedJackpotTickets = unclaimedCount[drawId][winningTicket[drawId]];
-            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
+            currentNetProfit = currentNetProfit + int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
         }
     }
```

## Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L181-L186
```solidity
File: /src/Lottery.sol

//@audit: uint128 drawId, uint120 ticket
181:    function registerTicket(
182:        uint128 drawId,
183:        uint120 ticket,
184:        address frontend,
185:        address referrer
186:    )

//@audit: uint8 winTier
234:    function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {

//@audit: uint128 drawId, uint8 winTier
238:    function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {
```


https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L23
```solidity
File: /src/Ticket.sol

//@audit: uint128 drawId, uint120 combination
23:    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L52-L57
```solidity
File: /src/ReferralSystem.sol

//@audit:  uint128 currentDraw
52:    function referralRegisterTickets(
53:        uint128 currentDraw,
54:        address referrer,
55:        address player,
56:        uint256 numberOfTickets
57:    )

//@audit: uint128 drawFinalized
87:    function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {

//@audit: uint128 drawId
136:    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {

//@audit: uint128 drawId
156:    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

//@audit: uint128 drawId
161:    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L17-L21
```solidity
File: /src/TicketUtils.sol

//@audit:  uint8 selectionSize,uint8 selectionMax
17:    function isValidTicket(
18:        uint256 ticket,
19:        uint8 selectionSize,
20:        uint8 selectionMax
21:    )

//@audit: uint8 selectionSize,uint8 selectionMax
43:    function reconstructTicket(
44:        uint256 randomNumber,
45:        uint8 selectionSize,
46:        uint8 selectionMax
47:    )

//@audit: uint120 ticket,uint120 winningTicket,uint8 selectionSize,uint8 selectionMax
83:    function ticketWinTier(
84:        uint120 ticket,
85:        uint120 winningTicket,
86:        uint8 selectionSize,
87:        uint8 selectionMax
88:    )
```

## Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L273
```solidity
File: /src/Lottery.sol
273:            uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
```
The operation `currentDraw - LotteryMath.DRAWS_PER_YEAR` cannot underflow due to the check on [Line 272](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L272) which ensures that `currentDraw` is greater than or equal to `LotteryMath.DRAWS_PER_YEAR`

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L158
```solidity
File: /src/ReferralSystem.sol
158:        return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;
```
The operation `playerRewardFirstDraw - decrease` would never underflow as it is only executed if `playerRewardFirstDraw` is greater than `decrease`

