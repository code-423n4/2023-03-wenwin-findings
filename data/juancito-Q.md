# QA Report

## Low Issues

### [L-1] Chainlink won't retry requests to get the winner ticket if the `fulfillRandomWords` function reverts

As stated on the Chainlink [documentation](https://docs.chain.link/vrf/v2/security#fulfillrandomwords-must-not-revert):

> If your `fulfillRandomWords()` implementation reverts, the VRF service will not attempt to call it a second time. Make
> sure your contract logic does not revert.

#### Proof of Concept

```solidity
File: src/VRFv2RNSource.sol

32:    function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {
33:        if (randomWords.length != 1) {
34:            revert WrongRandomNumberCountReceived(requestId, randomWords.length); // @audit should not revert
35:        }
36:
37:        fulfill(requestId, randomWords[0]); // @audit-info calls another function that reverts
38:    }
```

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L32-L38)

```solidity
File: src/RNSourceBase.sol

33:    function fulfill(uint256 requestId, uint256 randomNumber) internal {
34:        if (requests[requestId].status == RequestStatus.None) {
35:            revert RequestNotFound(requestId); // @audit should not revert
36:        }
37:        if (requests[requestId].status == RequestStatus.Fulfilled) {
38:            revert RequestAlreadyFulfilled(requestId); // @audit should not revert
39:        }
40:
41:        requests[requestId].randomNumber = randomNumber;
42:        requests[requestId].status = RequestStatus.Fulfilled;
43:        IRNSourceConsumer(authorizedConsumer).onRandomNumberFulfilled(randomNumber); // @audit-info calls another function that reverts
44:
45:        emit RequestFulfilled(requestId, randomNumber);
46:    }
```

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L33-L46)

```solidity
File: src/RNSourceController.sol

46:    function onRandomNumberFulfilled(uint256 randomNumber) external override {
47:        if (msg.sender != address(source)) {
48:            revert RandomNumberFulfillmentUnauthorized(); // @audit should not revert
49:        }
50:
51:        lastRequestFulfilled = true;
52:        failedSequentialAttempts = 0;
53:        maxFailedAttemptsReachedAt = 0;
54:
55:        receiveRandomNumber(randomNumber);
56:    }
```

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#LL46-L56C6)

#### Recommended Mitigation Steps

Refactor the code to prevent reverts and fail gracefully

### [L-2] Users can claim referral rewards without referring anyone if < 100 tickets are sold

#### Proof of Concept

This is due to a loss of precision when totalTicketsSoldPrevDraw < 100

```solidity
    function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
        internal
        view
        virtual
        returns (uint256 minimumEligible)
    {
        if (totalTicketsSoldPrevDraw < 10_000) {
            // 1%
            // @audit loss of precision for totalTicketsSoldPrevDraw < 100
            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT);
        }
        if (totalTicketsSoldPrevDraw < 100_000) {
            // 0.75%
            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);
        }
        if (totalTicketsSoldPrevDraw < 1_000_000) {
            // 0.5%
            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);
        }
        return 5000;
    }
```

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L111-L130)

#### Recommended Mitigation Steps

Return some minimum number of referrals to be elegible if tickets sold < 100

```diff
    function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
        internal
        view
        virtual
        returns (uint256 minimumEligible)
    {
+        if (totalTicketsSoldPrevDraw < 100) {
+            return minimumEligibleReferrals;
+        }
        if (totalTicketsSoldPrevDraw < 10_000) {
            // 1%
            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT);
        }
        if (totalTicketsSoldPrevDraw < 100_000) {
            // 0.75%
            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);
        }
        if (totalTicketsSoldPrevDraw < 1_000_000) {
            // 0.5%
            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);
        }
        return 5000;
    }
```

### [L-3] Use SafeCast for casting

From [OpenZeppelin documentation](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast):

> Downcasting from uint256/int256 in Solidity does not revert on overflow. This can easily result in undesired
> exploitation or bugs, since developers usually assume that overflows raise errors. SafeCast restores this intuition by
> reverting the transaction when such an operation overflows.

#### Proof of Concept

```solidity
    function returnUnclaimedJackpotToThePot() private {
        if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {
            uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
            uint256 unclaimedJackpotTickets = unclaimedCount[drawId][winningTicket[drawId]];
            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]); // @audit use SafeCast
        }
    }
```

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L271-L277)

````solidity
    function calculateNewProfit(
        int256 oldProfit,
        uint256 ticketsSold,
        uint256 ticketPrice,
        bool jackpotWon,
        uint256 fixedJackpotSize,
        uint256 expectedPayout
    )
        internal
        pure
        returns (int256 newProfit)
    {
        uint256 ticketsSalesToPot = (ticketsSold * ticketPrice).getPercentage(TICKET_PRICE_TO_POT);
        newProfit = oldProfit + int256(ticketsSalesToPot); // @audit use SafeCast

        uint256 expectedRewardsOut = jackpotWon
            ? calculateReward(oldProfit, fixedJackpotSize, fixedJackpotSize, ticketsSold, true, expectedPayout)
            : calculateMultiplier(calculateExcessPot(oldProfit, fixedJackpotSize), ticketsSold, expectedPayout)
                * ticketsSold * expectedPayout;

        newProfit -= int256(expectedRewardsOut); // @audit use SafeCast
    }
```solidity

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L35-L56)

```solidity
    function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot) {
        int256 excessPotInt = netProfit.getPercentageInt(SAFETY_MARGIN);
        excessPotInt -= int256(fixedJackpotSize); // @audit use SafeCast
        excessPot = excessPotInt > 0 ? uint256(excessPotInt) : 0;
    }
````

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L62-L66)

```solidity
    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
        return number * int256(percentage) / int256(PERCENTAGE_BASE); // @audit use SafeCast
    }
```

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L22-L24)

#### Recommended Mitigation Steps

Use SafeCast for casting

## Non Critical Issues

### [NC-1] Lack of Ticket NFTs metadata will make secondary markets more difficult to integrate

It is important for users to easily know and understand the kind of tickets they would be buying on secondary markets.
For example, they don't want to buy expired losing tickets, or they would prefer to know the numbers present on a ticket
for the upcoming draw.

This will help grow the secondary market economy.

#### Recommended Mitigation Steps

Consider adding NFT metadata that includes the `drawId`, the ticket number, the draw date, and any other relevant data
for secondary markets.

## Non Critical Issues

### [NC-2] DRAWS_PER_YEAR variable name is misleading

`DRAWS_PER_YEAR` is defined as `52`, but can be misleading, as `drawPeriod` can be initialized with any value.

```solidity
    function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier) {
        TicketInfo memory ticketInfo = ticketsInfo[ticketId];
        if (!ticketInfo.claimed) {
            uint120 _winningTicket = winningTicket[ticketInfo.drawId];
            winTier = TicketUtils.ticketWinTier(ticketInfo.combination, _winningTicket, selectionSize, selectionMax);
            // @audit DRAWS_PER_YEAR might be misleading, as the calculation depends on drawPeriod being one week
            if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {
                claimableAmount = winAmount[ticketInfo.drawId][winTier];
            }
        }
    }
```

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L159-L168)

```solidity
drawPeriod = lotterySetupParams.drawSchedule.drawPeriod;
```

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L84)

#### Recommended Mitigation Steps

Set the `drawPeriod` variable as immutable and define it as it corresponds with 52 draws per year.

Alternatively, change the variable name `DRAWS_PER_YEAR` to `DRAWS_PER_ROUND`, or some other name. But it will have
impact on the inflation, as the original calculation was thought to be yearly.

### [NC-3] Make proper use of NatSpec

Provide all the necessary information on the function NatSpec description, as it helps navigation on the code and self
description. Additionally add references like "See `getPercentage`."

#### Proof of Concept

```solidity
    /// @dev Calculates percentage of signed number. See `getPercentage`.
    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
        return number * int256(percentage) / int256(PERCENTAGE_BASE);
    }
```

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L21-L24)

#### Recommended Mitigation Steps

Provide sufficient NatSpec definitions for functions implementing it

Additionally include NatSpec definitions for all public facing functions

[NatSpec reference](https://docs.soliditylang.org/en/latest/style-guide.html#natspec)

### [NC-4] Misleading getPercentage function name

Getters should be used to get information, and not modify state

#### Proof of Concept

```solidity
    // @audit misleading function name
    function getReward() public override {
        _updateReward(msg.sender); // @audit modifies state
        uint256 reward = rewards[msg.sender];
        if (reward > 0) {
            rewards[msg.sender] = 0;
            // slither-disable-next-line unused-return
            lottery.claimRewards(LotteryRewardType.STAKING); // @audit modifies state
            rewardsToken.safeTransfer(msg.sender, reward); // @audit modifies state
            emit RewardPaid(msg.sender, reward);
        }
    }
```

[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L91-L101)

#### Recommended Mitigation Steps

Consider renaming the function to other name, like `claimReward`