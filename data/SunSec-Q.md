## [L1] Without max value validation cause out of gas
## Impact
One common cause of running out of gas is a missing or inadequate maximum value validation.

## Proof of Concept
```
function buyTickets(
    uint128[] calldata drawIds,
    uint120[] calldata tickets,
    address frontend,
    address referrer
)
    external
    override
    requireJackpotInitialized
    returns (uint256[] memory ticketIds)
{
    if (drawIds.length != tickets.length) { 
        revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
    }
    ticketIds = new uint256[](tickets.length);
    for (uint256 i = 0; i < drawIds.length; ++i) { //@audit drawIds without max value check
        ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer); 
```

Test: try to mint over 270,000 tickets. we will hit out of gas.
```
buySameTickets(currentDraw, uint120(0x0F), address(0), 270_000);

Encountered 1 failing test in test/ReferralSystemBase.sol:ReferralSystemBase
[FAIL. Reason: EvmError: OutOfGas] testAddTicket() (gas: 9223372036854754743)
```
## Tools Used
Manual
## Recommended Mitigation Steps
Add maximum value check for drawIds and tickets.


## [L2] No zero value checks for calculateRewards & dueTicketsSoldAndReset
## Impact

Lack of zero value check on both calculateRewards and dueTicketsSoldAndReset to transfer 0 amount to user.
Follow best practice is good to add additional value check.

## Proof of Concept
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L119-L129
```
//dueRewards without zaro amount check
    function calculateRewards(
        uint256 ticketPrice,
        uint256 ticketsSold,
        LotteryRewardType rewardType
    )
        internal
        pure
        returns (uint256 dueRewards)
    {
        uint256 rewardPercentage = (rewardType == LotteryRewardType.FRONTEND) ? FRONTEND_REWARD : STAKING_REWARD;
        dueRewards = (ticketsSold * ticketPrice).getPercentage(rewardPercentage);
    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L249-L255
```
//dueTickets without zaro ticket check
    function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {
        if (beneficiary == stakingRewardRecipient) {
            dueTickets = nextTicketId - claimedStakingRewardAtTicketId;
            claimedStakingRewardAtTicketId = nextTicketId;
        } else {
            dueTickets = frontendDueTicketSales[beneficiary];
            frontendDueTicketSales[beneficiary] = 0;
```
```
//If FRONTEND with zero reward, the tx will return success.
    ├─ [10009] Lottery::claimRewards(0) 
    │   ├─ emit ClaimedRewards(rewardRecipient: LotteryTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496], amount: 0, rewardType: 0)
    │   ├─ [5138] TestToken::transfer(LotteryTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496], 0) 
    │   │   ├─ emit Transfer(from: Lottery: [0x2e234DAe75C793f67A35089C9d99245E1C58470b], to: LotteryTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496], value: 0)
    │   │   └─ ← true

```
## Tools Used
Manual
## Recommended Mitigation Steps
Add a require() or additional check 

## [L3] Missing onlyOwner modifier in StakedTokenLock

## Impact
getReward without onlyOwner modifier which means eveyone can call function getReward().

## Proof of Concept
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L50-L55
```
function getReward() external override { //without onlyOwner
        stakedToken.getReward();

        // No need for SafeTransfer, only trusted reward token is used.
        // slither-disable-next-line unchecked-transfer
        rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));
    }
```

Although stakedToken.getReward() in getReward(); is to use msg.sender to claimRewards. This will not cause any fund loss.

But follow the line: it will transfer reward token to owner.
` rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));`

So it's not a good practice.
 
## Tools Used
Manual
## Recommended Mitigation Steps
Add onlyOwner modifier 


 