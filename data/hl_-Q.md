# Table of contents

- [N-01] Insufficient coverage
- [N-02] NatSpec comments should be increased in contracts
- [N-03] File does not contain an SPDX identifier
- [N-04] Solidity complier version too new
- [N-05] Exact functions not imported
- [L-01] `safemint()` should be used rather than `mint()`
- [L-02] Missing check in `First draw schedule` to ensure time not set in the past 
- [L-03] Check for zero address or amount == 0 in `LotterySetupParams` constructor
- [L-04] Avoid writing division before multiplication
- [L-05] Unbounded Loop when buying an excessive amount of tickets


## [N-01] Insufficient coverage
The test coverage rate of the project is 80.69%. Testing all functions is best practice in terms of security criteria.

## [N-02] NatSpec comments should be increased in contracts
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the [Solidity official documentation](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html).

## [N-03] File does not contain an SPDX identifier
Missing first line of code

For example: 
```
File:    2023-03-wenwin/src/Lottery.sol 
1:    // SPDX-License-Identifier: UNLICENSED
```
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L1

## [N-04] Solidity complier version too new
The complier version used in the contract is 0.8.19 which is very new and hence not yet thoroughly tested for possible bugs, etc. It is best practice to use a more thoroughly tested complier version e.g. 0.8.16

For example: 

```
File:    2023-03-wenwin/src/Lottery.sol 
3:    pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L1

## [N-05] Exact functions not imported
It is best practice for exact functions to be imported. 

For example: 

```
File:    2023-03-wenwin/src/Lottery.sol 
5:    import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
6:    import "@openzeppelin/contracts/utils/math/Math.sol";
7:    import "src/ReferralSystem.sol";
8:    import "src/RNSourceController.sol";
9:    import "src/staking/Staking.sol";
10:    import "src/LotterySetup.sol";
11:    import "src/TicketUtils.sol";
```
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L5-L11

## [L-01] `safemint()` should be used rather than `mint()` 

`mint()` is discouraged in favor of `safeMint()` which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin and solmate have versions of this function.

```
File:    2023-03-wenwin/src/Lottery.sol 
192:      ticketId = mint(msg.sender, drawId, ticket);
```
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L192

## [L-02] Missing check in `First draw schedule` to ensure time not set in the past 

Per the constructor of `LotterySetup.sol`, protocol owner sets the first draw schedule:

```
File:    2023-03-wenwin/src/LotterySetup.sol 
83:    firstDrawSchedule = lotterySetupParams.drawSchedule.firstDrawScheduledAt;
```

There should be a check for `firstDrawSchedule` > `block.timestamp` to ensure that `firstDrawSchedule` is not set in the past by mistake. 

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L83

## [L-03] Check for zero address or amount == 0 in `LotterySetupParams` constructor

It is best practice for every variable to be checked for 0 value in case of a typo error. 

```
File:     2023-03-wenwin/src/LotterySetup.sol 
86:    ticketPrice = lotterySetupParams.ticketPrice;
87:    selectionSize = lotterySetupParams.selectionSize;
88:    selectionMax = lotterySetupParams.selectionMax;
89:    expectedPayout = lotterySetupParams.expectedPayout;
```
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L86-L89

## [L-04] Avoid writing division before multiplication

```
File:    2023-03-wenwin/src/ReferralSystem.sol
122:    // 0.75%
123:    return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);
124:    }
125:    if (totalTicketsSoldPrevDraw < 1_000_000) {
126:    // 0.5%
127:    return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);
128:    } 
```
Write as the following instead:
`totalTicketsSoldPrevDraw.getPercentage(750)`
`totalTicketsSoldPrevDraw.getPercentage(500)`

## [L-05] Unbounded Loop when buying an excessive amount of tickets

The contract should be capable to take a large purchase from whales without reverting due to excessive gas. 

The `buyTickets()` function in `Lottery.sol` loops through the `drawId` length, which refers to the number of tickets that a player wants to buy.

If `drawIds.length` is too high due to the same player buying a massive number of tickets at one go, the function will revert due to reaching maximum gas limit. Consider breaking up the loop into smaller loops so that each execution can be successful.

```
File:    2023-03-wenwin/src/Lottery.sol 
124:    ticketIds = new uint256[](tickets.length);
125:    for (uint256 i = 0; i < drawIds.length; ++i) {
            ticketIds[i] = registerTicket(drawIds[i], 126:    tickets[i], frontend, referrer);
127:    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L124-L127
