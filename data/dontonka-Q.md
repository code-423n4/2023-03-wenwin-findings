Some changes were already made, tests are still happy with those.

```diff
diff --git a/src/Lottery.sol b/src/Lottery.sol
index 28d3477..833ea1a 100644
--- a/src/Lottery.sol
+++ b/src/Lottery.sol
@@ -161,7 +161,7 @@ contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceCont
         if (!ticketInfo.claimed) {
             uint120 _winningTicket = winningTicket[ticketInfo.drawId];
             winTier = TicketUtils.ticketWinTier(ticketInfo.combination, _winningTicket, selectionSize, selectionMax);
-            if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {
+            if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) { //@audit (QA) documentation claims ticket holder can claim until 1 year, but ticketRegistrationDeadline does substract drawCoolDownPeriod, so in reality its a little less then one year.
                 claimableAmount = winAmount[ticketInfo.drawId][winTier];
             }
         }
```

```diff
diff --git a/src/LotteryMath.sol b/src/LotteryMath.sol
index e3dee3e..b5a92d4 100644
--- a/src/LotteryMath.sol
+++ b/src/LotteryMath.sol
@@ -111,7 +111,7 @@ library LotteryMath {
             : fixedReward.getPercentage(calculateMultiplier(excess, ticketsSold, expectedPayout));
     }

-    /// @dev Calculate frontend rewards amount for specific tickets sold
+    /// @dev Calculate frontend rewards amount for specific tickets sold //@audit (QA) comment inacurate, its also for the staking rewards.
     /// @param ticketPrice One lottery ticket price
     /// @param ticketsSold Amount of tickets sold since last fee payout
     /// @param rewardType Type of the reward we are calculating
```

```diff
diff --git a/src/LotterySetup.sol b/src/LotterySetup.sol
index 9c70086..026f2c5 100644
--- a/src/LotterySetup.sol
+++ b/src/LotterySetup.sol
@@ -48,7 +48,7 @@ contract LotterySetup is ILotterySetup {
         if (lotterySetupParams.selectionSize == 0) {
             revert SelectionSizeZero();
         }
-        if (lotterySetupParams.selectionMax >= 120) {
+        if (lotterySetupParams.selectionMax >= 120) { //@audit (QA) should this not be > only as you use uint120, for what i understand you desire a maximum of 120 selection, not 119.
             revert SelectionSizeMaxTooBig();
         }
         if (
@@ -162,7 +162,7 @@ contract LotterySetup is ILotterySetup {
     }

     function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {
-        if (rewards.length != (selectionSize) || rewards[0] != 0) {
+        if (rewards.length != selectionSize || rewards[0] != 0) { //@audit (QA) parenthesis around selectionSize are useless.
             revert InvalidFixedRewardSetup();
         }
         uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1);
```

```diff
diff --git a/src/LotteryToken.sol b/src/LotteryToken.sol
index 9de87d3..ca2ec7b 100644
--- a/src/LotteryToken.sol
+++ b/src/LotteryToken.sol
@@ -9,12 +9,12 @@ import "src/LotteryMath.sol";
 /// @dev Lottery token contract. The token has a fixed initial supply.
 /// Additional tokens can be minted after each draw is finalized. Inflation rates (per draw) are defined for each year.
 contract LotteryToken is ILotteryToken, ERC20 {
-    uint256 public constant override INITIAL_SUPPLY = 1_000_000_000e18;
+    uint256 public constant override INITIAL_SUPPLY = 1_000_000_000e18; //@audit (QA) i don't understand how override here is relevant, its confusing at least, would suggest to remove.

     address public immutable override owner;

     /// @dev Initializes lottery token with `INITIAL_SUPPLY` pre-minted tokens
-    constructor() ERC20("Wenwin Lottery", "LOT") {
+    constructor() ERC20("Wenwin Lottery", "LOT") { // @audit (QA) The team desire is to use this code base for multiple game (not only the initial lottery game), but the fact that token name and symbol are hardcoded will make this very confusing. The team should have those value configurable also, so that the code base doesn't need to be modified at all in such case.
         owner = msg.sender;
         _mint(msg.sender, INITIAL_SUPPLY);
     }
```

```diff
diff --git a/src/RNSourceController.sol b/src/RNSourceController.sol
index 2bd8cfc..1a43939 100644
--- a/src/RNSourceController.sol
+++ b/src/RNSourceController.sol
@@ -24,10 +24,10 @@ abstract contract RNSourceController is Ownable2Step, IRNSourceController {
     /// it is removed from the list of sources
     /// @param _maxRequestDelay The maximum delay between random number request and its fulfillment
     constructor(uint256 _maxFailedAttempts, uint256 _maxRequestDelay) {
-        if (_maxFailedAttempts > MAX_MAX_FAILED_ATTEMPTS) {
+        if (_maxFailedAttempts > MAX_MAX_FAILED_ATTEMPTS) { //@audit (QA) I would also require to be greater than 0, as a random generator source, no lottery or future game you want to add should require disabling this robustness feature.
             revert MaxFailedAttemptsTooBig();
         }
-        if (_maxRequestDelay > MAX_REQUEST_DELAY) {
+        if (_maxRequestDelay > MAX_REQUEST_DELAY) { //@audit (QA) idem. the combination of those 2 could "disable" the robustess of the RN.
             revert MaxRequestDelayTooBig();
         }
         maxFailedAttempts = _maxFailedAttempts;
@@ -87,7 +87,7 @@ abstract contract RNSourceController is Ownable2Step, IRNSourceController {
     }

     function swapSource(IRNSource newSource) external override onlyOwner {
-        if (address(newSource) == address(0)) {
+        if (address(newSource) == address(0)) { //@audit (QA) code duplication: would probably do a modifier to re-use the code for this and the same code in initSource.
             revert RNSourceZeroAddress();
         }
         bool notEnoughRetryInvocations = failedSequentialAttempts < maxFailedAttempts;
```

```diff
diff --git a/src/ReferralSystem.sol b/src/ReferralSystem.sol
index d74ced7..3b2669f 100644
--- a/src/ReferralSystem.sol
+++ b/src/ReferralSystem.sol
@@ -114,15 +114,16 @@ abstract contract ReferralSystem is IReferralSystem {
         virtual
         returns (uint256 minimumEligible)
     {
-        if (totalTicketsSoldPrevDraw < 10_000) {
+        //@audit (QA) aligning with documentation the conditions
+        if (totalTicketsSoldPrevDraw <= 10_000) {
             // 1%
             return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT);
         }
-        if (totalTicketsSoldPrevDraw < 100_000) {
+        if (totalTicketsSoldPrevDraw <= 100_000) {
             // 0.75%
             return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);
         }
-        if (totalTicketsSoldPrevDraw < 1_000_000) {
+        if (totalTicketsSoldPrevDraw <= 1_000_000) {
             // 0.5%
             return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);
         }
@@ -136,16 +137,15 @@ abstract contract ReferralSystem is IReferralSystem {
     function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
         requireFinishedDraw(drawId);

-        UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
+        UnclaimedTicketsData storage _unclaimedTickets = unclaimedTickets[drawId][msg.sender]; //@audit (QA) Personally i would use storage here instead of memory as there is a chance to have a write and would make the code cleaner, without much impact really on GAS.
         if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {
             claimedReward = referrerRewardPerDrawForOneTicket[drawId] * _unclaimedTickets.referrerTicketCount;
-            unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
+            _unclaimedTickets.referrerTicketCount = 0;
         }

-        _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
         if (_unclaimedTickets.playerTicketCount > 0) {
             claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
-            unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
+            _unclaimedTickets.playerTicketCount = 0;
         }

         if (claimedReward > 0) {
```