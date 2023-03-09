```diff
diff --git a/src/ReferralSystem.sol b/src/ReferralSystem.sol
index d74ced7..74dad6f 100644
--- a/src/ReferralSystem.sol
+++ b/src/ReferralSystem.sol
@@ -59,14 +79,16 @@ abstract contract ReferralSystem is IReferralSystem {
     {
         if (referrer != address(0)) {
             uint256 minimumEligible = minimumEligibleReferrals[currentDraw];
-            if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
-                if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
-                    totalTicketsForReferrersPerDraw[currentDraw] +=
-                        unclaimedTickets[currentDraw][referrer].referrerTicketCount;
+
+            //@audit (GAS) caching the struct save nice gas.. also as storage as we might write to it.
+            UnclaimedTicketsData storage unclaimedTicketsCached = unclaimedTickets[currentDraw][referrer];
+            if (unclaimedTicketsCached.referrerTicketCount + numberOfTickets >= minimumEligible) {
+                if (unclaimedTicketsCached.referrerTicketCount < minimumEligible) {
+                    totalTicketsForReferrersPerDraw[currentDraw] += unclaimedTicketsCached.referrerTicketCount;
                 }
                 totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
             }
-            unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
+            unclaimedTicketsCached.referrerTicketCount += uint128(numberOfTickets);
         }
         unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
     }
```

```diff
diff --git a/src/Lottery.sol b/src/Lottery.sol
index 28d3477..dc0af26 100644
--- a/src/Lottery.sol
+++ b/src/Lottery.sol
@@ -104,7 +104,7 @@ contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceCont
             )
         );

-        nativeToken.safeTransfer(msg.sender, ILotteryToken(address(nativeToken)).INITIAL_SUPPLY());
+        nativeToken.safeTransfer(msg.sender, ILotteryToken(address(nativeToken)).INITIAL_SUPPLY()); //@audit (GAS) This seems useless as when creating the native token you already mint the initial supply to the msg.sender
```