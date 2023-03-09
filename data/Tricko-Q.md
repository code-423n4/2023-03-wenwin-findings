## Fee-on-transfer ERC20.
Fee-on-transfer tokens work by deducting a percentage of the transaction amount as a fee
However, `Lottery`'s transfer logic doesn't handle this type of token correctly, leading to loss of funds and wrong protocol functionality. 

If fee-on-transfer tokens are supposed to be accepted in the future, consider checking the token receiver balance before and after the transfer, and return the actual amount of tokens transferred to be used in the contract internal logic.

## No need to instantiate `_unclaimedTickets` twice.
In `ReferralSystem`'s `claimPerDraw` _unclaimedTickets doesn't need to be instantiated a second time. Although it is `unclaimedTickets[drawId][msg.sender]` is modified ealier, only the field `referrerTicketCount` is modified, therefore the field `.playerTicketCount` is still valid. Modification shown in the diff below.


```diff
diff --git a/ReferralSystem.orig.sol b/ReferralSystem.sol.
index d74ced7..a3ac663 100644
--- a/ReferralSystem.orig.sol
+++ b/ReferralSystem.sol
@@ -142,7 +142,6 @@ abstract contract ReferralSystem is IReferralSystem {
             unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
         }

-        _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
         if (_unclaimedTickets.playerTicketCount > 0) {
             claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
             unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
```

## Restrict users from adding themselves as `referrer` or `frontend`.
When buying tickets, users that aren't sending transactions through any frontend, can set `referrer` and `frontend` to their own address, this can make them claim more `LOT` as a normal user buying tickets should. Consider checking if `referrer` or `frontend` are equal to `msg.sender` and revert in this case. 

```diff
diff --git a/Lottery.orig.sol b/Lottery.sol
index 7da6d64..7d16a6e 100644
--- a/Lottery.orig.sol
+++ b/Lottery.sol
@@ -121,6 +121,7 @@ contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceCont
         if (drawIds.length != tickets.length) {
             revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
         }
+        require(frontend != msg.sender || referrer != msg.sender);
         ticketIds = new uint256[](tickets.length);
         for (uint256 i = 0; i < drawIds.length; ++i) {
             ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
```

## Fix NatSpec in `RNSourceBase`
`fulfill()` natspec references `requestRandomnessFromUnderlyingSource`, but the actual function should be `fulfillRandomWords`. As shown in the diff below.

```diff
diff --git a/RNSourceBase.orig.sol b/RNSourceBase.sol
index 6a8a445..2026fa6 100644
--- a/RNSourceBase.orig.sol
+++ b/RNSourceBase.sol
@@ -29,7 +29,7 @@ abstract contract RNSourceBase is IRNSource {
         emit RequestedRandomNumber(msg.sender, requestId);
     }

-    /// @dev Must be called in the deriving contract's `requestRandomnessFromUnderlyingSource` function
+    /// @dev Must be called in the deriving contract's `fulfillRandomWords` function
     function fulfill(uint256 requestId, uint256 randomNumber) internal {
         if (requests[requestId].status == RequestStatus.None) {
             revert RequestNotFound(requestId);
```