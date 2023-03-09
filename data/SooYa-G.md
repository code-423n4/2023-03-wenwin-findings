# Modifier That Used Only One Time Can Be Moved Inside Function

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

Modifier can save gas usage if it use in some function. But if it only used in one function, instead of using a modifier, its better to move the check inside the function.
In the Lottery.sol there are 4 modifier that only used once. So i recommend to remove the modifier, and add the check inside the function.

### Proof of Concept

Before removing the modifier
```
Running 2 tests for test/LotteryInvariantChecks.t.sol:LotteryInvariantChecksTest
[PASS] invariantSufficientFunds() (runs: 256, calls: 3840, reverts: 2629)
[PASS] testBuyClaimFinalize(uint256[],address[],uint256) (runs: 256, μ: 17494104, ~: 18119031)
Test result: ok. 2 passed; 0 failed; finished in 15.39s
╭──────────────────────────────────┬─────────────────┬───────┬────────┬────────┬─────────╮
│ src/Lottery.sol:Lottery contract ┆                 ┆       ┆        ┆        ┆         │
╞══════════════════════════════════╪═════════════════╪═══════╪════════╪════════╪═════════╡
│ Deployment Cost                  ┆ Deployment Size ┆       ┆        ┆        ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 8226018                          ┆ 39045           ┆       ┆        ┆        ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
```

After the modifier is removed
```
Running 2 tests for test/LotteryInvariantChecks.t.sol:LotteryInvariantChecksTest
[PASS] invariantSufficientFunds() (runs: 256, calls: 3840, reverts: 2672)
[PASS] testBuyClaimFinalize(uint256[],address[],uint256) (runs: 256, μ: 17626488, ~: 17956143)
Test result: ok. 2 passed; 0 failed; finished in 15.50s
╭──────────────────────────────────┬─────────────────┬────────┬────────┬────────┬─────────╮
│ src/Lottery.sol:Lottery contract ┆                 ┆        ┆        ┆        ┆         │
╞══════════════════════════════════╪═════════════════╪════════╪════════╪════════╪═════════╡
│ Deployment Cost                  ┆ Deployment Size ┆        ┆        ┆        ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 8225618                          ┆ 39043           ┆        ┆        ┆        ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
```

As you can see above, the deployment cost is decreasing about 400 gas.



# `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables. There are 2 instance in Lottery.sol:

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L129
```frontendDueTicketSales[frontend] += tickets.length;```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L173
```claimedAmount += claimWinningTicket(ticketIds[i]);```

Other issue:
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L43
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L78
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L96

### Proof of Concept
Before the change
```
Running 2 tests for test/LotteryInvariantChecks.t.sol:LotteryInvariantChecksTest
[PASS] invariantSufficientFunds() (runs: 256, calls: 3840, reverts: 2629)
[PASS] testBuyClaimFinalize(uint256[],address[],uint256) (runs: 256, μ: 17494104, ~: 18119031)
Test result: ok. 2 passed; 0 failed; finished in 15.39s
╭──────────────────────────────────┬─────────────────┬───────┬────────┬────────┬─────────╮
│ src/Lottery.sol:Lottery contract ┆                 ┆       ┆        ┆        ┆         │
╞══════════════════════════════════╪═════════════════╪═══════╪════════╪════════╪═════════╡
│ Deployment Cost                  ┆ Deployment Size ┆       ┆        ┆        ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 8226018                          ┆ 39045           ┆       ┆        ┆        ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤

```
After the change
```
Running 2 tests for test/LotteryInvariantChecks.t.sol:LotteryInvariantChecksTest
[PASS] invariantSufficientFunds() (runs: 256, calls: 3840, reverts: 2634)
[PASS] testBuyClaimFinalize(uint256[],address[],uint256) (runs: 256, μ: 17369419, ~: 16967222)
Test result: ok. 2 passed; 0 failed; finished in 14.82s
╭──────────────────────────────────┬─────────────────┬───────┬────────┬────────┬─────────╮
│ src/Lottery.sol:Lottery contract ┆                 ┆       ┆        ┆        ┆         │
╞══════════════════════════════════╪═════════════════╪═══════╪════════╪════════╪═════════╡
│ Deployment Cost                  ┆ Deployment Size ┆       ┆        ┆        ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
```
As you can see above, the deployment cost is decreasing about 1200 gas