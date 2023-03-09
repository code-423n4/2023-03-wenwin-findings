# Summary
|ID     | Finding|  Gas saved | Instances|
|:----: | :---           |  :----:         |:----:         |
|G-01      |Add unchecked block when operands can't under/overflow| 816 |  7 |
|G-02      |Use ERC721A for Ticket| - | 1 |
|G-03      |Remove return statements when it's never used| - | 2 |
|G-04      |Timestamps don't need to be bigger than uint48| - | 1 |
|G-05      |No 0 check on `claimedReward`| 10459 | 1 |
|G-06      |Unnecessary storage read| - | 1 |
|G-07      |Send staking rewards to staker directly| 21129 | 1 |
|G-08      |Check if dueTickets is 0| - | 1 |
# Details
## [G-01] Add unchecked block when operands can't under/overflow
When operands can't underflow/overflow because of a previous require() or if statement, you can use an unchecked block.
The default "checked" behavior costs more gas when calculating, because under-the-hood the EVM does extra checks for over/underflow.
|Code     |Explanation| Function |  Gas saved |
|:----: | :---           |  :----:         |:----:         |
|[LotterySetup.sol#L153](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L153)|All constant values that can't be manipulated and drawId only gets incremented by one| `executeDraw()` |404 |
|[LotterySetup.sol#L157](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L157)|Same as above| `claimable()` |412 |

This can also be done for divisions.

The expression type(int).min / (-1) is the only case where division causes an overflow:
- [LotterySetup.sol#L170](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L170)

And when the value is only incremented by 1:
- [Lottery.sol#L193-L194](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L193-L194)
- [Lottery.sol#L205](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L205)
- [Ticket.sol#L24](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L24)
- [TicketUtils.sol#L70](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L70)

## [G-02] Use ERC721A for Ticket
Using ERC721A is a lot more gas efficiënt when you buy multiple tickets. It will be a lot of refactoring work but definitely worth it. The amount of gas that can be saved can be seen [here](https://github.com/chiru-labs/ERC721A#about-the-project). When you use ERC721A you basically pay the same price if you buy 1 or 5 tickets. When you buy 1 or 5 tickets with the normal ERC721 it will be a huge difference. The more tickets you buy, the more you will save. When a big player wants to buy 100 tickets at once, the gas difference will be huge.

## [G-03] Remove return statements when it's never used
Both [buyTickets()](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L119) and [registerTicket()](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L181-L195) return the TicketId but this is never used. 
## [G-04] Timestamps don't need to be bigger than uint48
The maximum value of uint48 is year 8 million 921 thousand 556. It will be year 36,812 when it overflows. So the struct [`LotteryDrawSchedule`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L53-L60) can be packed in one storage slot.
## [G-05] No 0 check on `claimedReward`
When you call the function [`claimReferralReward()`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L76-L82) and the claimedReward is 0. You will still do the whole minting process of the native token even though it doesn't do anything and will mint 0 tokens.

When someone calls this function and they have 0 rewards, 10459 gas can be saved with the following if statement.
```diff
    function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {
        for (uint256 counter = 0; counter < drawIds.length; ++counter) {
            claimedReward += claimPerDraw(drawIds[counter]);
        }
+       if(claimedReward>0){
             mintNativeTokens(msg.sender, claimedReward);
+       }
    }
```
## [G-06] Unnecessary storage read
The storage variable [`unclaimedTickets[drawId][msg.sender]`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L145) gets saved twice to the same memory variable. Only `referrerTicketCount` gets set to 0 and it's not used afterwards so it's unneccessary to read from storage again. `_unclaimedTickets` can be used without updating it again.
```diff
        UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {
            claimedReward = referrerRewardPerDrawForOneTicket[drawId] * _unclaimedTickets.referrerTicketCount;
            unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
        }

-       _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.playerTicketCount > 0) {
            claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
            unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
        }
```
## [G-07] Send staking rewards to staker directly
The current implementation sends the rewardsTokens first to the stakingContract and afterwards to the staker. As you can see in [`getReward()`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L91-L101), it first calls [`claimRewards()`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L151-L157) in Lottery.sol which sends the funds to the staking contract. 

In most cases the rewardToken can be send directly to the staker. For this gas saver, I created a new function. This gas optimization works when people claim staking rewards after one or more tickets are bought. When people claim staking rewards multiple times and not tickets are bought in the meantime then there will be no gas saved. This is because Lottery.sol calculates the `claimedAmount` that gets send to the stakingContract with the following calculation `dueTickets = nextTicketId - claimedStakingRewardAtTicketId` in the function [`dueTicketsSoldAndReset()`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L249-L257). So when no extra tickets are sold between 2 staking reward claims, `dueTickets` will be 0. When that happens we can't send the rewardToken directly to the staker but we have to send it first to the StakingContract like in the current implementation.

An example:
1. Ticket bought
2. claim staking rewards: gas saved
3. Ticket bought
4. claim staking rewards: gas saved
5. claim staking rewards: no gas saved

21129 gas is saved when reward is sent directly to the staker.

I believe more tickets will be bought than the amount of times people will claim there staking rewards, so the gas optimization will have a big effect.

I made a new function similar to [`claimRewards()`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L151-L157) so it has no effect on the frontend rewards:
```solidity
    function claimRewardsStaking(address receiver, uint256 reward) external returns (bool directTransfer) {
        //check if msg.sender is Staking contract
        require(msg.sender == stakingRewardRecipient);
        LotteryRewardType rewardType = LotteryRewardType.STAKING;
        uint256 dueTickets = dueTicketsSoldAndReset(msg.sender);

        //if dueTickets is 0, claimedAmount will also be 0 and will transfer 0 tokens. Avoid this by returning false;
        if (dueTickets == 0) {
            return false;
        }
        uint256 claimedAmount = LotteryMath.calculateRewards(ticketPrice, dueTickets, rewardType);

        //When claimAmount is not equal to reward we have to send to the Staking first so it can transfer the tokens to the staker afterwards.
        if (claimedAmount != reward) {
            receiver = msg.sender;
        } else {
            //else we return true so the staking contract knows funds are transferred to the staker directly
            directTransfer = true;
        }

        emit ClaimedRewards(msg.sender, claimedAmount, rewardType);
        rewardToken.safeTransfer(receiver, claimedAmount);
    }
```
[`getReward()`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L91-L101) can be changed to:
```solidity
    function getReward() public override {
        _updateReward(msg.sender);
        uint256 reward = rewards[msg.sender];
        if (reward > 0) {
            rewards[msg.sender] = 0;
            
            bool directTransfer = lottery.claimRewardsStaking(msg.sender, reward);
            // if reward is not transferred directly, we send the reward now
            if (!directTransfer) {
                rewardsToken.safeTransfer(msg.sender, reward);
            }

            emit RewardPaid(msg.sender, reward);
        }
    }
```

## [G-08] Check if dueTickets is 0
If the optimization above is too complicated or the project believes there will be a lot of claiming staking rewards when no extra tickets are sold. Then there can be an if check in function [`claimRewards`](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L151-L157) to check the return of `dueTicketsSoldAndReset(beneficiary)` and see if dueTickets is 0. Because now, even if there are no extra tickets sold, which means 0 reward tokens will be send to the stakingRewardRecipient, it still does the whole process of calculating the reward and transferring 0 tokens.

The best way is to return 0 so it doesn't calculate the reward and send 0 tokens. If dueTickets is 0, claimedAmount will also be 0.
```diff
    function claimRewards(LotteryRewardType rewardType) external override returns (uint256 claimedAmount) {
        address beneficiary = (rewardType == LotteryRewardType.FRONTEND) ? msg.sender : stakingRewardRecipient;
+       uint256 dueTickets = dueTicketsSoldAndReset(beneficiary);
+       if(dueTickets==0) return 0;
-       claimedAmount = LotteryMath.calculateRewards(ticketPrice, dueTicketsSoldAndReset(beneficiary), rewardType);
+       claimedAmount = LotteryMath.calculateRewards(ticketPrice, dueTickets, rewardType);
        emit ClaimedRewards(beneficiary, claimedAmount, rewardType);
        rewardToken.safeTransfer(beneficiary, claimedAmount);
    }
```