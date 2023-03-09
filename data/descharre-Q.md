# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|L-01       |If not enough raised, project will not work as intended|1|
|L-02       |No limit on how many tickets you can buy in one transaction|1|
|L-03       |No 0 check on parameters|1|
|L-04       |NetProfit is not always correct because only jackpot winners are taken into consideration | 1 |


## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|N-01       | Don't use the latest solidity version | 1|

# Details
## Low Risk
## [L-01] If not enough raised, project will not work
In the unlikely scenario where the project won't sell enough tokens with their initial raise. The function `finalizeInitialPotRaise()` will revert and `initialPot` will stay 0 and this will have some consequences. This way when someone wins the jackpot they will receive a reward of 0 because the `initialPot` is 0.

[LotterySetup.sol#L141-L143](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L141-L143)
```solidity
        uint256 raised = rewardToken.balanceOf(address(this));
        if (raised < minInitialPot) {
            revert RaisedInsufficientFunds(raised);
        }
```
## [L-02] No limit on how many tickets you can buy in one transaction
Currently there is no limit on how many tickets you can buy. [`buyTickets()`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L110-L131) is an expensive function. The price of one ticket isn't very high so there can always be a big player that buys a lot of tickets so he has of a bigger chance of winning the jackpot. The more nfts you buy, the more expensive the transaction gets. 

When you buy around 550 tickets, which is 825 dai (usd). A big player is definitely willing to bet this kind of money. Then the function cost will be around 30 million gas which is the block gas limit. If the amount of gas exceed the gas limit, the transaction will revert and the big player will lose a lot of money in gas. Although this will not be this large if the lottery is run on the Polygon blockchain.

A mitigation is to have in if statement in `buyTickets()` to check that the length of `tickets` is not bigger than for example 500.
## [L-03] No 0 check on parameters
The parameters `_playerRewardFirstDraw` and `_playerRewardDecreasePerDraw` in [ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol) can be set to 0. In that case the playerRewards will not work and always be 0. Add a 0 check to be sure they can't be set accidently to 0.
## [L-04] NetProfit is not always correct because only jackpot winners are taken into consideration
Currently the net profit is calculated in [`calculateReward`()`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L96-L112) with an expected payout. However if there are more winners than expected and this happens multiple times. The NetProfit can be totally wrong after a few months or years.

## Non critical
## [N-01] Don't use the latest solidity version
The latest version of solidity might have some bugs that we don't know about yet. Consider using 0.8.18 or 0.8.17.
