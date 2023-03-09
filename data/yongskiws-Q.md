## Summary

### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-1]| Lack of Input Validation | 4 |
|[L-2]| Loss of precision due to rounding | 2 |

### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01]|Use require instead of assert|2|
| [N-02]|NatSpec comments should be increased in contracts|ALL CONTRACT|
| [N-03]|For modern and more readable code; update import usages|ALL CONTRACT|
| [N-04]|Function writing that does not comply with the Solidity Style Guide|ALL CONTRACT|
| [N-05]|Include return parameters in NatSpec comments|ALL CONTRACT|


### [L-01] Lack of Input Validation

Lack of input validation is the process of checking input data to ensure that it is valid and safe to process. Therefore, it is important for contracts to consider the lack of input validation to ensure that what is produced is secure and protected from potential security vulnerabilities. This can be achieved by ensuring that every user input is thoroughly checked and validated at every level of data processing, from user interface to database. By doing so, the application can be reliable and protected from security threats that may arise due to lack of input validation.

## Recommended Mitigation Steps

``` solidity

Lottery.sol
110:  function buyTickets(
111:         uint128[] calldata drawIds,
112:         uint120[] calldata tickets,
113:         address frontend,
114:         address referrer
115:     )
116:         external
117:         override
118:         requireJackpotInitialized
119:         returns (uint256[] memory ticketIds)
120:     {
121:         if (drawIds.length != tickets.length) {
122:             revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
123:         }
+        uint256 totalTicketPrice = ticketPrice * tickets.length;
+        require(rewardToken.balanceOf(msg.sender) >= totalTicketPrice, "Insufficient balance");
124:         ticketIds = new uint256[](tickets.length);
125:         for (uint256 i = 0; i < drawIds.length; ++i) {
126:             ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
127:         }
128:         referralRegisterTickets(currentDraw, referrer, msg.sender, tickets.length);
129:         frontendDueTicketSales[frontend] += tickets.length;
130:         rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);
131:     }

 
Lottery.sol
135:     function executeDraw() external override whenNotExecutingDraw {
136:         // slither-disable-next-line timestamp
137:         if (block.timestamp < drawScheduledAt(currentDraw)) {
138:             revert ExecutingDrawTooEarly();
139:         }
+            require(ticketSales > 0, "no tickets sold");
140:         returnUnclaimedJackpotToThePot();
141:         drawExecutionInProgress = true;
142:         requestRandomNumber();
143:         emit StartedExecutingDraw(currentDraw);
144:     }


Lottery.sol
function unclaimedRewards(LotteryRewardType rewardType) external view override returns (uint256 rewards) {
+    uint256 dueTicketsSold;
+    if (rewardType == LotteryRewardType.FRONTEND) {
+        dueTicketsSold = frontendDueTicketSales[msg.sender];
+    } else {
+        uint256 unclaimedTickets = nextTicketId - claimedStakingRewardAtTicketId;
+        require(unclaimedTickets > 0, "No unclaimed tickets for staking rewards");
+        uint256 ticketsOwned = ticketIdsOwnedByAddress[msg.sender].length;
+        require(ticketsOwned > 0, "No tickets owned by address for staking rewards");
+        dueTicketsSold = unclaimedTickets > ticketsOwned ? ticketsOwned : unclaimedTickets;
+    }
+    rewards = LotteryMath.calculateRewards(ticketPrice, dueTicketsSold, rewardType);
}

Lottery.sol
151: function claimRewards(LotteryRewardType rewardType) external override returns (uint256 claimedAmount) {
152:         address beneficiary = (rewardType == LotteryRewardType.FRONTEND) ? msg.sender : stakingRewardRecipient;
+153:         uint256 unclaimedRewards = LotteryMath.calculateRewards(ticketPrice, dueTicketsSoldAndReset(beneficiary),      rewardType);
+154:         require(unclaimedRewards > 0, "No unclaimed rewards available.");
155:         
+156:         claimedAmount = unclaimedRewards;
157:         emit ClaimedRewards(beneficiary, claimedAmount, rewardType);
158:         rewardToken.safeTransfer(beneficiary, claimedAmount);
159: }
```



### [L-02] Loss of precision due to rounding


``` solidity
LotteryMath.sol
85:         if (excessPot > 0 && ticketsSold > 0) {
86:             bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
87:         }


PercentageMath.sol
17:     function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
18:         return number * percentage / PERCENTAGE_BASE;
19:     }
20: 
21:     /// @dev Calculates percentage of signed number. See `getPercentage`.
22:     function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
23:         return number * int256(percentage) / int256(PERCENTAGE_BASE);
24:     }
```


### [N-01] Use require instead of assert

Description: Assert should not be used except for tests, require should be used

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process's available gas instead of returning it, as request()/revert() did.

assert() and ruqire(); The big difference between the two is that the assert()function when false, uses up all the remaining gas and reverts all the changes made. Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states "The Assert function generates an error of type Panic(uint256).Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there's a mistake".


``` solidity
LotterySetup.sol
147:         assert(initialPot > 0);
TicketUtils.sol
99:  assert((winTier <= selectionSize) && (intersection == uint256(0)));
``` 

### [N-02] NatSpec comments should be increased in contracts
Context: All Contracts

Description: It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Recommendation: NatSpec comments should be increased in contracts

### [N-03] For modern and more readable code; update import usages
Description: Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

## Recommendation
import {contract1 , contract2} from "filename.sol";

### [N-04] Function writing that does not comply with the Solidity Style Guide

Context: All Contracts

Description: Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor receive function (if exists) fallback function (if exists) external public internal private within a grouping, place the view and pure functions last


### [N-05] Include return parameters in NatSpec comments

Context: All Contracts

Description: It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## Recommendation
Include return parameters in NatSpec comments

``` diff
+/// @return add comment for return parameters
Lottery.sol
236:     function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
237:         return drawRewardSize(currentDraw, winTier);
238:     }
```
