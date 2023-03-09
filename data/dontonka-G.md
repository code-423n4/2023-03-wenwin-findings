That's the single worth GAS optimization I found which was not listed in the known issues. I made the changed already, tests are happy and GAS reduced.

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