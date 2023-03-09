# Low Risk Issues

## Summary Of Findings:

|  | Issue 
-- | -- 
1 | In the early stages users ends up receiving very little rewards because of massive locked stake from the protocol-team
2 | unsafe revert in `fulfillRandomWords` function
3 | Mismatch between documentation and smart contract in `getMinimumEligibleReferralsFactorCalculation` function
4 | Wrong formula for calculating Excess Pot
5 | The window for claiming winning tickets closes before 1 year
6 | The first draw should start only after the 200 million team tokens are staked and locked for 1 year

## Details on Findings:

### 1. <ins>In the early stages users ends up receiving very little rewards because of massive locked stake from the protocol-team</ins>

During the initial token launch, out of the 1 billion tokens, 20% of it(which is 200m), is allocated to team in one go. Though this is locked for 1 year, its still staked in the `Staking` contract and is eligible for a share of rewards. This is fine, if during the initial sale, the rest of the 750m are bought by users and they stake it in the Staking contract as well. 

But generally, in the initial stages, the number of native tokens in the hand of regular users will be much less than 200m. Which means a huge share of rewards will go to the protocol team. 

So, to make it fairer to regular users, It is recommended that, the protocol team doesnt get all the 200m tokens at once. One way would be to use a simple formula, like this:
`Tokens allocated to team = min(200m, Tokens sold to regular users)`

This will let the team grow in a linear way with the rest of the users. 

### 2. <ins>unsafe revert in `fulfillRandomWords` function</ins>

[fulfillRandomWords]() function in `VRFv2RNSource.sol` has an unsafe revert which can make the whole lottery system stuck.
According to Chainlink security guidelines, this function must never revert, as once called it will not get called again. So a revert will totally brick the system and lead to massive loss of funds.

Reference - https://docs.chain.link/vrf/v2/security/#fulfillrandomwords-must-not-revert

Though you shouldn't receive more than one word from Chainlink, we recommended to just use the first word instead of reverting if there are multiple words.

### 3. <ins>Mismatch between documentation and smart contract in `getMinimumEligibleReferralsFactorCalculation` function</ins>

The [getMinimumEligibleReferralsFactorCalculation](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L111) function uses a series of `if` conditions to decide the `minimumEligible` referral amount based on which range `totalTicketsSoldPrevDraw` falls into. There is a small difference here, between the documentation and code.

The [doc](https://docs.wenwin.com/wenwin-lottery/protocol-architecture/token/rewards/referrals) uses `<=` sign where code uses `<`. Either the doc should be corrected or the code. 

### 4. <ins>Wrong formula for calculating Excess Pot</ins>

[calculateExcessPot](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L62-L66) function in `LotteryMath` uses a somewhat different formula for calculating the Excess Pot than the one in the documentation does. This will result in discrepancies if not set up correctly.
```solidity
    function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot) {
        int256 excessPotInt = netProfit.getPercentageInt(SAFETY_MARGIN);
        excessPotInt -= int256(fixedJackpotSize);
        excessPot = excessPotInt > 0 ? uint256(excessPotInt) : 0;
    }
```

In short, the formula in code boils down to:
```
excessPotInt = netProfit * SAFETY_MARGIN - fixedJackpotSize;
```

But the [actual formula](https://docs.wenwin.com/wenwin-lottery/the-game/prizes/bonuses) from the doc shows:
```
excessPotInt = netProfit * (1-SAFETY_MARGIN) - fixedJackpotSize;
```

Submitting it as a QA, only because, the developers might have stored `1-SAFETY_MARGIN` as `SAFETY_MARGIN` in the code. Otherwise this is a med or high severity.

### 5. <ins>The window for claiming winning tickets closes before 1 year</ins>

The [documentation](https://docs.wenwin.com/wenwin-lottery/the-game/prizes#claiming-prizes) says that:
> Winning tickets have a one-year deadline from the draw time to claim their prize by redeeming the winning NFT ticket. 

But this is not implemented in the code correctly. The time period is a bit less than 1 year. The [claimable](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L159) function checks if the current timestamp is less than or equal to `ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)`.

The `ticketRegistrationDeadline` function returns `firstDrawSchedule + (drawId * drawPeriod) - drawCoolDownPeriod;`. 

Assuming each draw lasts for 7 days and 52 draws per year, this contributes to 7*52 = 360 days. Which is at least 5 days less than an year. Further, **the `drawCoolDownPeriod` is subtracted from the timestamp**, which means deadline for claiming the winning tickets, end at least drawCoolDownPeriod before 1 year period. 

It is recommended to mention this fact in the doc for transparency. Or the code could be modified from:
` if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {`

to

` if (block.timestamp <= drawScheduledAt(ticketInfo.drawId) + 1 year) {`

### 6. <ins>The first draw should start only after the 200 million team tokens are staked and locked for 1 year</ins>

The [documentation](https://docs.wenwin.com/wenwin-lottery/protocol-architecture/token) says 20% of the native tokens will be allocated to the team which is around 200 million in this case. These tokens are supposed to be locked for an year. But this condition is not enforced anywhere in the code.  

We recommend that the [finalizeInitialPotRaise()](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L132) function should make sure that these team tokens are indeed staked and locked before finalizing the Pot. This will ensure added relief for the dear early investors and give more credibility to the protocol. 



