## Summary

## Before Test Coverege

| src/Lottery.sol:Lottery contract |                 |        |        |        |         |
|----------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                  | Deployment Size |        |        |        |         |
| 8226018                          | 39045           |        |        |        |         |
| Function Name                    | min             | avg    | median | max    | # calls |
| buyTickets                       | 2187            | 113113 | 104028 | 531848 | 175     |
| claimRewards                     | 8159            | 42361  | 51959  | 51959  | 11      |
| claimWinningTickets              | 1009            | 15371  | 16054  | 31165  | 6       |
| claimable                        | 794             | 2941   | 2960   | 4960   | 167     |
| currentDraw                      | 504             | 695    | 504    | 2504   | 167     |
| executeDraw                      | 467             | 8753   | 6045   | 51156  | 114     |
| finalizeInitialPotRaise          | 2440            | 23670  | 24520  | 24520  | 26      |
| fixedReward                      | 527             | 1861   | 550    | 4506   | 3       |
| initSource                       | 24075           | 24075  | 24075  | 24075  | 25      |
| initialPotDeadline               | 263             | 263    | 263    | 263    | 27      |
| jackpotBound                     | 329             | 329    | 329    | 329    | 1       |
| nativeToken                      | 327             | 327    | 327    | 327    | 35      |
| nextTicketId                     | 385             | 626    | 385    | 2385   | 58      |
| onRandomNumberFulfilled          | 73740           | 98066  | 94951  | 160084 | 112     |
| stakingRewardRecipient           | 307             | 307    | 307    | 307    | 11      |
| ticketPrice                      | 330             | 330    | 330    | 330    | 35      |
| unclaimedRewards                 | 1107            | 1505   | 1172   | 3238   | 6       |
| winAmount                        | 749             | 749    | 749    | 749    | 2       |
| winningTicket                    | 649             | 649    | 649    | 649    | 2       |


| src/LotteryToken.sol:LotteryToken contract  |                 |       |        |       |         |
|---------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                             | Deployment Size |       |        |       |         |
| 607209                                      | 3378            |       |        |       |         |
| Function Name                               | min             | avg   | median | max   | # calls |
| INITIAL_SUPPLY                              | 240             | 240   | 240    | 240   | 28      |
| approve(address,uint256)                    | 24651           | 24651 | 24651  | 24651 | 1       |
| approve(address,uint256)(bool)              | 24651           | 24651 | 24651  | 24651 | 8       |
| balanceOf                                   | 585             | 585   | 585    | 585   | 5       |
| mint                                        | 427             | 25864 | 29568  | 29568 | 11      |
| totalSupply                                 | 349             | 1015  | 349    | 2349  | 3       |
| transfer                                    | 18413           | 19902 | 20013  | 20013 | 29      |
| transferFrom(address,address,uint256)       | 22143           | 22143 | 22143  | 22143 | 1       |
| transferFrom(address,address,uint256)(bool) | 4623            | 17763 | 22143  | 22143 | 8       |


| src/VRFv2RNSource.sol:VRFv2RNSource contract |                 |       |        |       |         |
|----------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                              | Deployment Size |       |        |       |         |
| 428653                                       | 2646            |       |        |       |         |
| Function Name                                | min             | avg   | median | max   | # calls |
| rawFulfillRandomWords                        | 887             | 8549  | 1296   | 23465 | 3       |
| requestRandomNumber                          | 28317           | 28317 | 28317  | 28317 | 1       |
    

| src/staking/StakedTokenLock.sol:StakedTokenLock contract |                 |       |        |       |         |
|----------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                          | Deployment Size |       |        |       |         |
| 464921                                                   | 2815            |       |        |       |         |
| Function Name                                            | min             | avg   | median | max   | # calls |
| deposit                                                  | 2518            | 33452 | 42286  | 42286 | 9       |
| getReward                                                | 21656           | 21656 | 21656  | 21656 | 1       |
| withdraw                                                 | 576             | 13443 | 19377  | 19443 | 6       |


| src/staking/Staking.sol:Staking contract |                 |        |        |        |         |
|------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                          | Deployment Size |        |        |        |         |
| 1177729                                  | 7012            |        |        |        |         |
| Function Name                            | min             | avg    | median | max    | # calls |
| approve                                  | 24692           | 24692  | 24692  | 24692  | 1       |
| earned                                   | 3861            | 3861   | 3861   | 3861   | 5       |
| exit                                     | 158339          | 158339 | 158339 | 158339 | 1       |
| getReward                                | 7922            | 76704  | 69234  | 132914 | 7       |
| stake                                    | 396             | 67487  | 72369  | 83950  | 10      |
| transfer                                 | 123780          | 123780 | 123780 | 123780 | 1       |
| transferFrom                             | 117113          | 117113 | 117113 | 117113 | 1       |
| withdraw                                 | 308             | 14730  | 14730  | 29153  | 2       |


### Gas Optimizations Issues List
| Number |Issues Details|Instance|
|:--:|:-------|:--:|
|[G-01]| Using unchecked blocks to save gas | 16 |
|[G-02]| Using unchecked blocks to save gas - Increments in for loop can be unchecked ( save 30-40 gas per loop iteration) | 8 |
|[G-03]| Using storage instead of memory for structs/arrays saves gas | 3 |
|[G-04]| fulfill(): requests[requestId] should be cached in local storage | 1 |
|[G-05]| Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate | 5 |
|[G-06]| Using private rather than public for constants, saves gas | ALL CONTRACT |
|[G-07]| Using immutable for uint & address for save gas | ALL CONTRACT |
|[G-08]| internal functions not called by the contract should be removed to save deployment gas | 3 |
|[G-09]| State variables can be packed into fewer storage slots  | 1 |

Total 40 issues

## [G-01] Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
https://github.com/ethereum/solidity/issues/10695

There are 16 instances of this issue:

The above should be modified to:
``` diff
Lottery.sol
+279:             unchecked {
280:             currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
+281:             }


Lottery.sol
+253:             unchecked {
254:             dueTickets = nextTicketId - claimedStakingRewardAtTicketId;
255:             claimedStakingRewardAtTicketId = nextTicketId;
+256:             }

Lottery.sol
+194:         unchecked {
195:         unclaimedCount[drawId][ticket]++;
196:         ticketsSold[drawId]++;
+197:         }

LotteryMath.sol
+84:             unchecked {
85:             bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
+86:             }


LotteryMath.sol
+130:         unchecked {
131:         uint256 rewardPercentage = (rewardType == LotteryRewardType.FRONTEND) ? FRONTEND_REWARD : STAKING_REWARD;
132:         dueRewards = (ticketsSold * ticketPrice).getPercentage(rewardPercentage);
+133:         }


LotterySetup.sol
120:     function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
121:         if (winTier == selectionSize) {
122:             return _baseJackpot(initialPot);
123:         } else if (winTier == 0 || winTier > selectionSize) {
124:             return 0;
125:         } else {
+126:             unchecked {
127:             uint256 mask = uint256(type(uint16).max) << (winTier * 16);
128:             uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
129:             return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
+130:             }
131:         }
132:     }


LotterySetup.sol
154:     function drawScheduledAt(uint128 drawId) public view override returns (uint256 time) {
+155:         unchecked {
156:         time = firstDrawSchedule + (drawId * drawPeriod);
157:         }
+158:     }
159: 


LotterySetup.sol
160:     function ticketRegistrationDeadline(uint128 drawId) public view override returns (uint256 time) {
+161:         unchecked {
162:         time = drawScheduledAt(drawId) - drawCoolDownPeriod;
+163:         }
164:     }


ReferralSystem.sol
52:    function referralRegisterTickets(
53:         uint128 currentDraw,
54:         address referrer,
55:         address player,
56:         uint256 numberOfTickets
57:     )
58:         internal
59:     {
60:         if (referrer != address(0)) {
61:             uint256 minimumEligible = minimumEligibleReferrals[currentDraw];
62:             if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
63:                 if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
+64:                     unchecked{
65:                     totalTicketsForReferrersPerDraw[currentDraw] +=
66:                         unclaimedTickets[currentDraw][referrer].referrerTicketCount;
+67:                     }
68:                 }
+69:                 unchecked{
70:                 totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
+71:                 }
72:             }
+73:             unchecked{
74:             unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
+75:             }
76:         }
+77:         unchecked{ 
78:         unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
+79:         }
80:     }
81: 



ReferralSystem.sol
87:     function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {
        // if no tickets sold there is no incentives, so no rewards to be set
        if (ticketsSoldDuringDraw == 0) {
            return;
        }

        minimumEligibleReferrals[drawFinalized + 1] =
            getMinimumEligibleReferralsFactorCalculation(ticketsSoldDuringDraw);

        uint256 referrerRewardForDraw = referrerRewardsPerDraw(drawFinalized);
        uint256 totalTicketsForReferrersPerCurrentDraw = totalTicketsForReferrersPerDraw[drawFinalized];
        if (totalTicketsForReferrersPerCurrentDraw > 0) {
+            unchecked{ 
            referrerRewardPerDrawForOneTicket[drawFinalized] =
                referrerRewardForDraw / totalTicketsForReferrersPerCurrentDraw;
+            }
        }

        uint256 playerRewardForDraw = playerRewardsPerDraw(drawFinalized);
        if (playerRewardForDraw > 0) {
+            unchecked{  
            playerRewardsPerDrawForOneTicket[drawFinalized] = playerRewardForDraw / ticketsSoldDuringDraw;
+            }
        }

        emit CalculatedRewardsForDraw(drawFinalized, referrerRewardForDraw, playerRewardForDraw);
    }


ReferralSystem.sol
136:     function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
        requireFinishedDraw(drawId);

        UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {
+            unchecked{
            claimedReward = referrerRewardPerDrawForOneTicket[drawId] * _unclaimedTickets.referrerTicketCount;
            unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
+            }
        }

        _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.playerTicketCount > 0) {
+            unchecked{
            claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
            unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
+            }
        }

        if (claimedReward > 0) {
            emit ClaimedReferralReward(drawId, msg.sender, claimedReward);
        }
    }


ReferralSystem.sol
156:     function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
+        unchecked{
        uint256 decrease = uint256(drawId) * playerRewardDecreasePerDraw;
        return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;           
+        }
    }
```

## Gas Saved For Using unchecked blocks to save gas
| src/Lottery.sol:Lottery contract |                 |       |        |        |         |
|----------------------------------|-----------------|-------|--------|--------|---------|
| Deployment Cost                  | Deployment Size |       |        |        |         |
| 8099250                          | 38412           |       |        |        |         |
| Function Name                    | min             | avg   | median | max    | # calls |
| buyTickets                       | 1959            | 85784 | 64754  | 529265 | 174     |
| claimRewards                     | 7822            | 42061 | 51622  | 51622  | 11      |
| claimWinningTickets              | 1009            | 15211 | 15849  | 30800  | 6       |
| claimable                        | 794             | 2715  | 2732   | 4732   | 167     |
| currentDraw                      | 504             | 696   | 504    | 2504   | 166     |
| executeDraw                      | 467             | 8624  | 5920   | 51000  | 114     |
| finalizeInitialPotRaise          | 2871            | 23687 | 24520  | 24520  | 26      |
| fixedReward                      | 527             | 1655  | 550    | 3889   | 3       |
| initSource                       | 24075           | 24075 | 24075  | 24075  | 25      |
| initialPotDeadline               | 263             | 263   | 263    | 263    | 27      |
| jackpotBound                     | 329             | 329   | 329    | 329    | 1       |
| minInitialPot                    | 262             | 262   | 262    | 262    | 1       |
| nativeToken                      | 327             | 327   | 327    | 327    | 35      |
| nextTicketId                     | 385             | 626   | 385    | 2385   | 58      |
| onRandomNumberFulfilled          | 73572           | 96993 | 93470  | 173169 | 112     |
| stakingRewardRecipient           | 307             | 307   | 307    | 307    | 11      |
| ticketPrice                      | 330             | 330   | 330    | 330    | 35      |
| unclaimedRewards                 | 937             | 1335  | 1002   | 3067   | 6       |
| winAmount                        | 749             | 749   | 749    | 749    | 2       |
| winningTicket                    | 649             | 649   | 649    | 649    | 2       |


| src/LotteryToken.sol:LotteryToken contract  |                 |       |        |       |         |
|---------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                             | Deployment Size |       |        |       |         |
| 607209                                      | 3378            |       |        |       |         |
| Function Name                               | min             | avg   | median | max   | # calls |
| INITIAL_SUPPLY                              | 240             | 240   | 240    | 240   | 28      |
| approve(address,uint256)                    | 24651           | 24651 | 24651  | 24651 | 2       |
| approve(address,uint256)(bool)              | 24651           | 24651 | 24651  | 24651 | 7       |
| balanceOf                                   | 585             | 585   | 585    | 585   | 5       |
| mint                                        | 427             | 25864 | 29568  | 29568 | 11      |
| totalSupply                                 | 349             | 1015  | 349    | 2349  | 3       |
| transfer                                    | 18413           | 19902 | 20013  | 20013 | 29      |
| transferFrom(address,address,uint256)       | 4623            | 13383 | 13383  | 22143 | 2       |
| transferFrom(address,address,uint256)(bool) | 4623            | 19640 | 22143  | 22143 | 7       |


| src/VRFv2RNSource.sol:VRFv2RNSource contract |                 |       |        |       |         |
|----------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                              | Deployment Size |       |        |       |         |
| 428653                                       | 2646            |       |        |       |         |
| Function Name                                | min             | avg   | median | max   | # calls |
| rawFulfillRandomWords                        | 887             | 8549  | 1296   | 23465 | 3       |
| requestRandomNumber                          | 28317           | 28317 | 28317  | 28317 | 1       |


| src/staking/StakedTokenLock.sol:StakedTokenLock contract |                 |       |        |       |         |
|----------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                          | Deployment Size |       |        |       |         |
| 464921                                                   | 2815            |       |        |       |         |
| Function Name                                            | min             | avg   | median | max   | # calls |
| deposit                                                  | 2518            | 33452 | 42286  | 42286 | 9       |
| getReward                                                | 21656           | 21656 | 21656  | 21656 | 1       |
| withdraw                                                 | 576             | 13443 | 19377  | 19443 | 6       |


| src/staking/Staking.sol:Staking contract |                 |        |        |        |         |
|------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                          | Deployment Size |        |        |        |         |
| 1172322                                  | 6985            |        |        |        |         |
| Function Name                            | min             | avg    | median | max    | # calls |
| approve                                  | 24692           | 24692  | 24692  | 24692  | 1       |
| earned                                   | 3690            | 3690   | 3690   | 3690   | 5       |
| exit                                     | 157796          | 157796 | 157796 | 157796 | 1       |
| getReward                                | 7580            | 76171  | 68691  | 132371 | 7       |
| stake                                    | 396             | 67433  | 72369  | 83676  | 10      |
| transfer                                 | 123096          | 123096 | 123096 | 123096 | 1       |
| transferFrom                             | 116566          | 116566 | 116566 | 116566 | 1       |
| withdraw                                 | 308             | 14594  | 14594  | 28880  | 2       |



## [G-02] Using unchecked blocks to save gas - Increments in for loop can be unchecked ( save 30-40 gas per loop iteration)

The majority of Solidity for loops increment a uint256 variable that starts at 0. These increment operations never need to be checked for over/underflow because the variable will never reach the max number of uint256 (will run out of gas long before that happens). The default over/underflow check wastes gas in every iteration of virtually every for loop . eg.

e.g Let's work with a sample loop below.
``` solidity
for(uint256 i; i < 10; i++){
//doSomething
}
```
can be written as shown below.
``` diff
for(uint256 i; i < 10;) {
  // loop logic
+  unchecked { i++; }
}
```
We can also write it as an inlined function like below.
``` solidity
function inc(i) internal pure returns (uint256) {
  unchecked { return i + 1; }
}
for(uint256 i; i < 10; i = inc(i)) {
  // doSomething
}
```

The above should be modified to:

There are 8 instances of this issue:

``` diff
Lottery.sol
125:         for (uint256 i = 0; i < drawIds.length;) {
126:             ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
+127:             unchecked {
+128:                 ++i;
+129:             }
130:         }

Other Instances to modify

Lottery.sol
175:         for (uint256 i = 0; i < totalTickets;) {
176:             claimedAmount += claimWinningTicket(ticketIds[i]);
+177:             unchecked {
+178:                 ++i;
+179:             }   
180:         }

ReferralSystem.sol
-35:         for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
36:             if (_rewardsToReferrersPerDraw[i] == 0) {
37:                 revert ReferrerRewardsInvalid();
38:             }
39:         }

TicketUtils.sol
-28:             for (uint8 i = 0; i < selectionMax; ++i) {
29:                 ticketSize += (ticket & uint256(1));
30:                 ticket >>= 1;
31:             }

TicketUtils.sol
56: 
-57:         for (uint256 i = 0; i < selectionSize; ++i) {
58:             numbers[i] = uint8(randomNumber % currentSelectionCount);
59:             randomNumber /= currentSelectionCount;
60:             currentSelectionCount--;
61:         }
62: 

TicketUtils.sol
-68:             for (uint256 j = 0; j <= currentNumber; ++j) {
69:                 if (selected[j]) {
70:                     currentNumber++;
71:                 }
72:             }

TicketUtils.sol
-68:             for (uint256 j = 0; j <= currentNumber; ++j) {
69:                 if (selected[j]) {
70:                     currentNumber++;
71:                 }
72:             }


TicketUtils.sol
-95:             for (uint8 i = 0; i < selectionMax; ++i) {
96:                 winTier += uint8(intersection & uint120(1));
97:                 intersection >>= 1;
98:             }
```

## [G-03] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


There are 3 instances of this issue:


``` solidity
ReferralSystem.sol
139:  UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
TicketUtils.sol
54:         uint8[] memory numbers = new uint8[](selectionSize);
TicketUtils.sol
63:         bool[] memory selected = new bool[](selectionMax);
```

## [G-04] fulfill(): requests[requestId] should be cached in local storage
There are 1 instances of this issue:
``` soliditiy
RNSourceBase.sol
41:       requests[requestId].randomNumber = randomNumber;
42:       requests[requestId].status = RequestStatus.Fulfilled;
```

## [G-05] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0.

There are 5 instances of this issue:

``` solidity
Lottery.sol
26:     mapping(address => uint256) private frontendDueTicketSales;
27:     mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
36:     mapping(uint128 => uint120) public override winningTicket;
37:     mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
39:     mapping(uint128 => uint256) public override ticketsSold;

ReferralSystem.sol
17:     mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;
19:     mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;
21:     mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;
23:     mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;
25:     mapping(uint128 => uint256) public override minimumEligibleReferrals;
9:      mapping(uint256 => RandomnessRequest) internal requests;

Staking.sol
19:     mapping(address => uint256) public override userRewardPerTokenPaid;
20:     mapping(address => uint256) public override rewards;

RNSourceBase.sol
9:     mapping(uint256 => RandomnessRequest) internal requests;

Ticket.sol
14:     mapping(uint256 => ITicket.TicketInfo) public override ticketsInfo;
```

## [G-06] Using private rather than public for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

There are ALL CONTRACT instances of this issue eg. :

``` diff
- uint256 public override example;
+ uint256 private override example;
```

## [G-07] Using immutable for uint & address for save gas
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

There are ALL CONTRACT instances of this issue eg. :

``` diff
- address public override example;
+ address public immutable override example;
```

## [G-08] internal functions not called by the contract should be removed to save deployment gas
If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

There are 3 instances of this issue:

LotterySetup.sol
160:     function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {
161:         return Math.min(_initialPot.getPercentage(BASE_JACKPOT_PERCENTAGE), jackpotBound);
162:     }

ReferralSystem.sol
156:     function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
157:         uint256 decrease = uint256(drawId) * playerRewardDecreasePerDraw;
158:         return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;
159:     }

ReferralSystem.sol
161:     function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
162:         return rewardsToReferrersPerDraw[Math.min(rewardsToReferrersPerDraw.length - 1, drawId)];
163:     }


## [G-09] State variables can be packed into fewer storage slots 

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper 

There are 1 instances of this issue:
``` diff
| src/Lottery.sol:Lottery contract |                 |        |        |        |         |
|----------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                  | Deployment Size |        |        |        |         |
| 8216201                          | 38996           |        |        |        |         |
| Function Name                    | min             | avg    | median | max    | # calls |
+| onRandomNumberFulfilled          | 74214           | 106163 | 100727 | 202043 | 112     |
```

``` diff
Lottery.sol
21: contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceController {
22:     using SafeERC20 for IERC20;
23:     using TicketUtils for uint256;
24: 
25:     uint256 private claimedStakingRewardAtTicketId;
26:     mapping(address => uint256) private frontendDueTicketSales;
27:     mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
+28:    bool public override drawExecutionInProgress;

29:     address public immutable override stakingRewardRecipient;
30: 
31:     uint256 public override lastDrawFinalTicketId;
32: 
-33:    bool public override drawExecutionInProgress;
34:     uint128 public override currentDraw;
```
