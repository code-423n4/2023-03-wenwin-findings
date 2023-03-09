# Low Severity Findings

## [L-01] No check on `lotterySetupParams.fixedRewardshaving` having all indexes in array up to selectionSize

- https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L91

If an admin accidentally configures a poor selection size, they would also have to update the fixed rewards, otherwise the reward tiers will be unfair. For example, if you update `SELECTION_SIZE` to 7 and `SELECTION_MAX` to 35 in `LotteryTestBase.sol` and then add this test to Lottery.t.sol

```javascript
function testPatrickFixedRewards() public {
        console.log(lottery.fixedReward(0)); // 0
        console.log(lottery.fixedReward(1)); // 5.000000000000000000
        console.log(lottery.fixedReward(2)); // 10.000000000000000000
        console.log(lottery.fixedReward(3)); // 15.000000000000000000
        console.log(lottery.fixedReward(4)); // 0
        console.log(lottery.fixedReward(5)); // 0
        console.log(lottery.fixedReward(6)); // 0
        console.log(lottery.fixedReward(7)); // 300,300.000000000000000000 jackpot
    }
```

### Mitigation

Run a require/check on deployment to see if all fixedRewards are accounted for based on the selection size.

## [L-02] An attacker can make `lastRequestFulfilled = false` by frontrunning `executeDraw()` before source is initialized

- https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L133

If an attacker front runs `RNSourceController.requestRandomNumber` before the source is initialized by admin, `lastRequestFulfilled` will be false in `requestRandomNumberFromSource()` and the contract would start working after waiting for 5 hours and calling `retry()`.

1. The `RNSourceController` was created by the admin.
2. An attacker front runs `executeDraw()` before the source is initialized.
3. Then `lastRequestFulfilled = false` in `requestRandomNumberFromSource()` and it won't revert as source.`requestRandomNumber()` is within try-catch
4. As `lastRequestFulfilled = false`, `requestRandomNumber()` will revert and it will be the same after the source is initialized by the admin because initSource() doesn't reset lastRequestFulfilled.
5. As a result, the contract should call `retry()` to start working after `maxRequestDelay`

### Mitigation

`initSource()` should set `lastRequestFulfilled = true`, `maxFailedAttempts = 0`, `maxRequestDelay = 0` like `swapSource()`

Otherwise, we would consider setting the source inside the constructor.

Or, we could have `executeDraw` not be callable unless source is initialized. 

## [L-03] Some percent of frontend rewards wouldn't be claimed forever for some cases and it should be added to net profit

- https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L113

In `buyTickets()`, it tracks `frontendDueTicketSales` for frontend and every frontend can claim rewards(10%) by calling `claimRewards()`.

But if `frontend == address(0)` in `buyTickets()`, the tickets will be added to `address(0)` and it won’t be claimed forever.

It means the rewards will be locked inside the contract as there is no logic to recover such rewards(like increasing net profit accordingly).

### Mitigation

Add a check to make sure `frontend != address(0)`


# Informational / Non-Crit Findings

## [I-01] Prizes are said to be both dynamic & static

- https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L20
- https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/ILotterySetup.sol#L76


In `Lottery.sol` it says:
```
/// All prizes are dynamic and dependant on the actual ticket sales.
```

Yet, in `ILotterySetup.sol` it clearly says:

```
// @dev Array of fixed rewards per each non jackpot win
```

It cannot be true that all prizes are dynamic yes rewards for non-jackpot wins are fixed.

### Mitigation

Update the comment in `Lottery.sol` to reflect that some prizes are fixed. 

## [I-02] Inflation rate is not correct based on what's in `ReferralSystemConfig.sol`

In the [wenwin docs](https://docs.wenwin.com/wenwin-lottery/protocol-architecture/token/rewards) it's said the inflation rate is a "hypothetical max" of 8% for the first year. 

With current parameters, the first week can be a max of `8.6%` as shown by the code below (if added to `Lottery.t.sol`):

```solidity
        function testInflationHigherThanAnticipated() public {
        uint256 startingLotSupply = lotteryToken.totalSupply();
        console.log("Starting supply: ", startingLotSupply);

        uint128 currentDraw = lottery.currentDraw();

        vm.startPrank(USER);
        uint256 amount = 500_000_000 ether;
        rewardToken.mint(amount); // DAI
        rewardToken.approve(address(lottery), amount);
        uint128 targetDrawId = currentDraw + 100_000_000; // 100_000_000 weeks away, doesn't matter though.
        uint256 drawIdsSize = 1;
        uint128[] memory drawIds = new uint128[](drawIdsSize);
        uint120[] memory tickets = new uint120[](drawIdsSize);
        for (uint256 i = 0; i < drawIdsSize; i++) {
            drawIds[i] = targetDrawId;
            tickets[i] = uint120(0x0F);
        }
        // User is going to get all the inflation LOT
        address frontend = USER;
        address referer = USER;
        lottery.buyTickets(drawIds, tickets, frontend, referer);
        vm.stopPrank();

        vm.warp(block.timestamp + 60 * 60 * 24);
        lottery.executeDraw();

        uint256 randomNumber = 0x00000000;
        vm.prank(randomNumberSource);
        lottery.onRandomNumberFulfilled(randomNumber);
        uint128[] memory targetDrawIds = new uint128[](1);
        targetDrawIds[0] = currentDraw;

        vm.startPrank(USER);
        lottery.claimReferralReward(targetDrawIds);
        // 1,661,538.500000000000000000 LOT
        vm.stopPrank();

        uint256 endingSupply = lotteryToken.totalSupply();

        // 8% inflation across 52 weeks is 0.153846153846154% of initial supply a week
        // 8 / 52 = 0.153846153846154
        assertFalse(endingSupply <= ((startingLotSupply * 153_846_153_846_154) / 1_000_000_000_000_000));
        console.log("Starting supply: ", startingLotSupply);
        console.log("Ending supply: ", endingSupply);
        console.log("Inflation: ", endingSupply - startingLotSupply); // 8.6%
        console.log("Expected inflation: ", (startingLotSupply * 153_846_153_846_154) / 1_000_000_000_000_000);
    }
```

## [I-03] Comment wrong for `reconstructTicket`

- https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/TicketUtils.sol#L36

The comment says it will divide by 2^8=256 but the implementation divides by total count of 35.