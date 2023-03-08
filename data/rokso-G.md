# Gas optimization report

There are multiple gas saving opportunities are found and all of these has impacted code linked, a description of issue and recommendation. I have also added git diff for recommendation for each issue.

**Total Findings = 12**

- [Use local variables](#use-local-variables)
  - [Store array length in local variable and use at multiple places](#store-array-length-in-local-variable-and-use-at-multiple-places)
  - [`referrerTicketCount` can be stored in local variable](#referrerticketcount-can-be-stored-in-local-variable)
  - [Request `status` can be stored in local variable](#request-status-can-be-stored-in-local-variable)
- [Check required condition before executing logic](#check-required-condition-before-executing-logic)
- [Unnecessary line of code](#unnecessary-line-of-code)
- [Reuse local variable](#reuse-local-variable)
- [Simplify `if` condition to save gas](#simplify-if-condition-to-save-gas)
- [Avoid unnecessary assignment to storage variable](#avoid-unnecessary-assignment-to-storage-variable)
- [Define new immutable variable `unlockTimestamp` and initialize in constructor](#define-new-immutable-variable-unlocktimestamp-and-initialize-in-constructor)
- [Unnecessary assert check](#unnecessary-assert-check)
- [Two for loops can be merged into one loop](#two-for-loops-can-be-merged-into-one-loop)
- [Unnecessary assignment to unused variable](#unnecessary-assignment-to-unused-variable)

## Use local variables

### Store array length in local variable and use at multiple places
  
> This is different than what automated tool has reported.

**Code:** [Lottery.sol#L121-L130](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L121-L130)

**Issue:** Linked code is using array length at multiple places in function and not just in `for loop`.

**Recommendation:** Consider storing length of `drawIds.length` in local variable and it can be used as replacement of `drawIds.length` and `tickets.length` after `if` condition.


```diff
diff --git a/src/Lottery.sol b/src/Lottery.sol
index 28d3477..5ae6fd9 100644
--- a/src/Lottery.sol
+++ b/src/Lottery.sol
@@ -118,16 +118,17 @@ contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceCont
         requireJackpotInitialized
         returns (uint256[] memory ticketIds)
     {
-        if (drawIds.length != tickets.length) {
-            revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
+        uint256 len = drawIds.length;
+        if (len != tickets.length) {
+            revert DrawsAndTicketsLenMismatch(len, tickets.length);
         }
-        ticketIds = new uint256[](tickets.length);
-        for (uint256 i = 0; i < drawIds.length; ++i) {
+        ticketIds = new uint256[](len);
+        for (uint256 i = 0; i < len; ++i) {
             ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
         }
-        referralRegisterTickets(currentDraw, referrer, msg.sender, tickets.length);
-        frontendDueTicketSales[frontend] += tickets.length;
-        rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);
+        referralRegisterTickets(currentDraw, referrer, msg.sender,len);
+        frontendDueTicketSales[frontend] += len;
+        rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * len);
     }
```

### `referrerTicketCount` can be stored in local variable
**Code:** [ReferralSystem.sol#L62-L66](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L62-L66)

**Issue:** Code is reading mapping storage value `unclaimedTickets[currentDraw][referrer].referrerTicketCount` multiple times.

**Recommendation:** Consider storing aforementioned storage variable in local variable and reference local variable at other places inside function.


```diff
diff --git a/src/ReferralSystem.sol b/src/ReferralSystem.sol
index d74ced7..fa59047 100644
--- a/src/ReferralSystem.sol
+++ b/src/ReferralSystem.sol
@@ -59,10 +59,10 @@ abstract contract ReferralSystem is IReferralSystem {
     {
         if (referrer != address(0)) {
             uint256 minimumEligible = minimumEligibleReferrals[currentDraw];
-            if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
-                if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
-                    totalTicketsForReferrersPerDraw[currentDraw] +=
-                        unclaimedTickets[currentDraw][referrer].referrerTicketCount;
+            uint256 referrerTicketCount = unclaimedTickets[currentDraw][referrer].referrerTicketCount;
+            if (referrerTicketCount + numberOfTickets >= minimumEligible) {
+                if (referrerTicketCount < minimumEligible) {
+                    totalTicketsForReferrersPerDraw[currentDraw] += referrerTicketCount;
                 }
                 totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
             }
```

### Request `status` can be stored in local variable

**Code:** [RNSourceBase.sol#L34-L39](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L34-L39)

**Issue:** For a successful, which will be the case mostly, call to `fulfill()` require reading request `status` multiple times from storage.

**Recommendation:** Consider storing requests[requestId].status in local variable.

```diff
diff --git a/src/RNSourceBase.sol b/src/RNSourceBase.sol
index 616fba8..5fb8fbc 100644
--- a/src/RNSourceBase.sol
+++ b/src/RNSourceBase.sol
@@ -31,10 +31,11 @@ abstract contract RNSourceBase is IRNSource {
 
     /// @dev Must be called in the deriving contract's `requestRandomnessFromUnderlyingSource` function
     function fulfill(uint256 requestId, uint256 randomNumber) internal {
-        if (requests[requestId].status == RequestStatus.None) {
+        RequestStatus status = requests[requestId].status;
+        if (status == RequestStatus.None) {
             revert RequestNotFound(requestId);
         }
-        if (requests[requestId].status == RequestStatus.Fulfilled) {
+        if (status == RequestStatus.Fulfilled) {
             revert RequestAlreadyFulfilled(requestId);
         }
```

## Check required condition before executing logic
**Code:** [LotterySetup.sol#L170-L173](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L170-L173)

**Issue:** Code is calculating reward and then checking `if ((rewards[winTier] % divisor) != 0)`. It may revert is condition is true and gas to calculate reward is wasted.

**Recommendation:** Consider moving `if` condition before calculating reward.

```diff
diff --git a/src/LotterySetup.sol b/src/LotterySetup.sol
index 9c70086..d8b11f4 100644
--- a/src/LotterySetup.sol
+++ b/src/LotterySetup.sol
@@ -167,10 +167,10 @@ contract LotterySetup is ILotterySetup {
         }
         uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1);
         for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {
-            uint16 reward = uint16(rewards[winTier] / divisor);
             if ((rewards[winTier] % divisor) != 0) {
                 revert InvalidFixedRewardSetup();
             }
+            uint16 reward = uint16(rewards[winTier] / divisor);
             packed |= uint256(reward) << (winTier * 16);
         }
     }
```

## Unnecessary line of code

**Code:** [ReferralSystem.sol#L145](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L145)

**Issue:** Code is reading `unclaimedTickets[drawId][msg.sender]` and based on some condition is it setting one of the struct param. Now code is reading `unclaimedTickets[drawId][msg.sender]` again which make sense if you are going to use updated param later which is not the case here. So there is no need to read storage variable again in this case.

**Recommendation:** Remove `_unclaimedTickets = unclaimedTickets[drawId][msg.sender]` line

```diff
diff --git a/src/ReferralSystem.sol b/src/ReferralSystem.sol
index d74ced7..a3ac663 100644
--- a/src/ReferralSystem.sol
+++ b/src/ReferralSystem.sol
@@ -142,7 +142,6 @@ abstract contract ReferralSystem is IReferralSystem {
             unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
         }
 
-        _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
         if (_unclaimedTickets.playerTicketCount > 0) {
             claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
             unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
```

## Reuse local variable

**Code:** [RNSourceController.sol#L73](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L73)

**Issue:** Storage variable `failedSequentialAttempts` is being used during emitting event. Before emitting event, code is updating storage variable and storing value in local variable `failedAttempts`.

**Recommendation:** Local variable `failedAttempts` can be used to replace storage variable when emitting event.

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


## Simplify `if` condition to save gas
**Code:** [RNSourceController.sol#L93-L95](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L93-L95)

**Issue:** There are 2 conditions which decides if logic has `notEnoughFailedAttempts`. `if` statement has 2 OR conditions and code is calculating both conditions before checking those in `if` but there is no need to calculate both condition before hand, as 2nd condition will only execute if 1st condition is `false`.

**Recommendation:** Use both condition expressions directly in `if` statement, doing this will save gas for not calculating 2nd condition unnecessarily and saves gas as results of expressions are not stored in local variables.

```diff
diff --git a/src/RNSourceController.sol b/src/RNSourceController.sol
index 2bd8cfc..7e5d524 100644
--- a/src/RNSourceController.sol
+++ b/src/RNSourceController.sol
@@ -90,9 +90,7 @@ abstract contract RNSourceController is Ownable2Step, IRNSourceController {
         if (address(newSource) == address(0)) {
             revert RNSourceZeroAddress();
         }
-        bool notEnoughRetryInvocations = failedSequentialAttempts < maxFailedAttempts;
-        bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;
-        if (notEnoughRetryInvocations || notEnoughTimeReachingMaxFailedAttempts) {
+        if ((failedSequentialAttempts < maxFailedAttempts) || (block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay)) {
             revert NotEnoughFailedAttempts();
         }
         source = newSource;
```

## Avoid unnecessary assignment to storage variable
**Code:** [RNSourceBase.sol#L27](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L27)

**Issue:** Code is initializing a `RandomnessRequest` struct with both, `status` and `randomNumber`, of it's params and storing it in storage but only `status` param needs update.

**Recommendation:** To avoid setting unnecessary params consider updating only required struct param directly compare to setting whole struct to storage.

```diff
diff --git a/src/RNSourceBase.sol b/src/RNSourceBase.sol
index 616fba8..5b22369 100644
--- a/src/RNSourceBase.sol
+++ b/src/RNSourceBase.sol
@@ -24,7 +24,7 @@ abstract contract RNSourceBase is IRNSource {
             revert requestIdAlreadyExists(requestId);
         }
 
-        requests[requestId] = RandomnessRequest({ status: RequestStatus.Pending, randomNumber: 0 });
+        requests[requestId].status = RequestStatus.Pending;
 
         emit RequestedRandomNumber(msg.sender, requestId);
     }
```

## Define new immutable variable `unlockTimestamp` and initialize in constructor
**Code:** [StakedTokenLock.sol#L39](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L39)

**Issue:** Withdraw() function has `if` condition which is checking `block.timestamp < depositDeadline + lockDuration`. depositDeadline and lockDuration both are storage and immutable variables. So it is possible to store sum of these 2 as another immutable variable.

**Recommendation:** Consider adding a new immutable variable which holds sum of `depositDeadline` and `lockDuration` and use this new variable in `if` condition in `withdraw()`.

```diff
diff --git a/src/staking/StakedTokenLock.sol b/src/staking/StakedTokenLock.sol
index f756d4d..58dfdb6 100644
--- a/src/staking/StakedTokenLock.sol
+++ b/src/staking/StakedTokenLock.sol
@@ -11,6 +11,7 @@ contract StakedTokenLock is IStakedTokenLock, Ownable2Step {
     IERC20 public immutable override rewardsToken;
     uint256 public immutable override depositDeadline;
     uint256 public immutable override lockDuration;
+    uint256 public immutable unlockTimestamp;
     uint256 public override depositedBalance;
 
     constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
@@ -19,6 +20,7 @@ contract StakedTokenLock is IStakedTokenLock, Ownable2Step {
         rewardsToken = stakedToken.rewardsToken();
         depositDeadline = _depositDeadline;
         lockDuration = _lockDuration;
+        unlockTimestamp = depositDeadline + lockDuration;
     }
 
     function deposit(uint256 amount) external override onlyOwner {
@@ -36,7 +38,7 @@ contract StakedTokenLock is IStakedTokenLock, Ownable2Step {
 
     function withdraw(uint256 amount) external override onlyOwner {
         // slither-disable-next-line timestamp
-        if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {
+        if (block.timestamp > depositDeadline && block.timestamp < unlockTimestamp) {
             revert LockPeriodOngoing();
         }
```

## Unnecessary assert check
**Code:** [LotterySetup.sol#L140-L147](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L140-L147)

**Issue:** `minInitialPot` is initialized as `minInitialPot = 4 * tokenUnit` during deployment of contract.
`assert(initialPot > 0)` check is not required because 
- initialPot = raised and 
- raised > minInitialPot and 
- minInitialPot is always > 0 (minInitialPot = 4 * tokenUnit;)

**Recommendation:** Remove `assert(initialPot > 0)` check.


```diff
diff --git a/src/LotterySetup.sol b/src/LotterySetup.sol
index 9c70086..2663af4 100644
--- a/src/LotterySetup.sol
+++ b/src/LotterySetup.sol
@@ -138,14 +138,12 @@ contract LotterySetup is ILotterySetup {
             revert FinalizingInitialPotBeforeDeadline();
         }
         uint256 raised = rewardToken.balanceOf(address(this));
+
         if (raised < minInitialPot) {
             revert RaisedInsufficientFunds(raised);
         }
         initialPot = raised;
 
-        // must hold after this call, this will be used as a check that jackpot is initialized
-        assert(initialPot > 0);
-
         emit InitialPotPeriodFinalized(raised);
     }
```


## Two for loops can be merged into one loop

**Code:** [TicketUtils.sol#L54-L75](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L54-L75)

**Issue:** There are two for loops in `reconstructTicket()` function. One loop is preparing an array `numbers` based on some calculations. Second loop is using looping over `numbers` and calculating elements for boolean array `selected`. It is possible to merge these two for loops into one as each iteration can calculate `currentNumber` and use it right away to calculate it is selected or not.

**Recommendation:** Consider merging two for loops into one.

```diff
diff --git a/src/TicketUtils.sol b/src/TicketUtils.sol
index be447aa..202ec45 100644
--- a/src/TicketUtils.sol
+++ b/src/TicketUtils.sol
@@ -51,19 +51,14 @@ library TicketUtils {
     {
         /// Ticket must contain unique numbers, so we are using smaller selection count in each iteration
         /// It basically means that, once `x` numbers are selected our choice is smaller for `x` numbers
-        uint8[] memory numbers = new uint8[](selectionSize);
+        bool[] memory selected = new bool[](selectionMax);
         uint256 currentSelectionCount = uint256(selectionMax);
 
         for (uint256 i = 0; i < selectionSize; ++i) {
-            numbers[i] = uint8(randomNumber % currentSelectionCount);
+            uint8 currentNumber = uint8(randomNumber % currentSelectionCount);
             randomNumber /= currentSelectionCount;
             currentSelectionCount--;
-        }
 
-        bool[] memory selected = new bool[](selectionMax);
-
-        for (uint256 i = 0; i < selectionSize; ++i) {
-            uint8 currentNumber = numbers[i];
             // check current selection for numbers smaller than current and increase if needed
             for (uint256 j = 0; j <= currentNumber; ++j) {
                 if (selected[j]) {
```

## Unnecessary assignment to unused variable
**Code:** [Lottery.sol#L260-L264](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L260-L264)

**Issue:** `claimWinningTicket()` function is calling `claimable` and storing returned variable into two local variables. Second variable is being assigned value but never used.

**Recommendation:** Consider removing local variable `winTier` and it's assignment.

```diff
diff --git a/src/Lottery.sol b/src/Lottery.sol
index 28d3477..14eb1b3 100644
--- a/src/Lottery.sol
+++ b/src/Lottery.sol
@@ -257,8 +257,7 @@ contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceCont
     }
 
     function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {
-        uint256 winTier;
-        (claimedAmount, winTier) = this.claimable(ticketId);
+        (claimedAmount,) = this.claimable(ticketId);
         if (claimedAmount == 0) {
             revert NothingToClaim(ticketId);
         }
```
