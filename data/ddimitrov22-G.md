  ## Summary

|        | Issues                                                                                                     | Contexts | Estimated Gas Saved |
| ------ | ---------------------------------------------------------------------------------------------------------- | -------- | ------------------- |
| [G-01] | Setting the constructor to `payable`                                                                       | 6        | 156                   |
| [G-02] | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables                                     | 16       | -                   |
| [G-03] | Empty blocks should be removed or emit something                                                           | 1        | -                   |
| [G-04] | Use `require` instead of `assert`                                                                       | 1        | -                   |
| [G-05] | Internal/Private functions only called once can be inlined to save gas                                                   | 9        | 180                   |
| [G-06] | Internal function is not called and can be removed          | 1       | -                   |


## [G-01] Setting the constructor to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor `payable` eliminates the need for an initial check of `msg.value == 0` and saves 13 gas on deployment with no security risks.

**Context**

    10 results - 10 files

    src/Lottery.sol:
    84:     constructor(

    src/LotterySetup.sol:
    41:     constructor(LotterySetupParams memory lotterySetupParams) {

    src/LotteryToken.sol:
    17:     constructor() ERC20("Wenwin Lottery", "LOT") {

    src/ReferralSystem.sol:
    27:     constructor(

    src/RNSourceBase.sol:
    11:     constructor(address _authorizedConsumer) {

    src/RNSourceController.sol:
    26:     constructor(uint256 _maxFailedAttempts, uint256 _maxRequestDelay) {

    src/Ticket.sol:
    17:     constructor() ERC721("Wenwin Lottery Ticket", "WLT") { }

    src/VRFv2RNSource.sol:
    13:     constructor(

    src/staking/StakedTokenLock.sol:
    16:     constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {

    src/staking/Staking.sol:
    22:     constructor(

## [G-02] `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

**Context**

    13 results - 5 files

    src/Lottery.sol:
    129:         frontendDueTicketSales[frontend] += tickets.length;
    173:             claimedAmount += claimWinningTicket(ticketIds[i]);
    275:             currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);

    src/LotteryMath.sol:
    84:             bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);

    src/ReferralSystem.sol:
    64:                     totalTicketsForReferrersPerDraw[currentDraw] +=
    67:                 totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
    69:             unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
    71:         unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
    78:             claimedReward += claimPerDraw(drawIds[counter]);
    147:             claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;

    src/TicketUtils.sol:
    29:                 ticketSize += (ticket & uint256(1));
    96:                 winTier += uint8(intersection & uint120(1));

    src/staking/StakedTokenLock.sol:
    30:         depositedBalance += amount;

    3 results - 2 files

    src/LotteryMath.sol:
    55:         newProfit -= int256(expectedRewardsOut);
    64:         excessPotInt -= int256(fixedJackpotSize);

    src/staking/StakedTokenLock.sol:
    43:         depositedBalance -= amount;



## [G-03] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be `abstract` and the function signatures be added without any default implementation.

    1 result - 1 file

    src/Ticket.sol:
    17:     constructor() ERC721("Wenwin Lottery Ticket", "WLT") { }

## [G-04] Use `require` instead of `assert`

The `assert()` and `require()` functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the `assert()` function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a `require()` function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

**Context**

    2 results - 2 files

    src/LotterySetup.sol:
    147:         assert(initialPot > 0);

    src/TicketUtils.sol:
    99:             assert((winTier <= selectionSize) && (intersection == uint256(0)));

## [G-05] Internal/Private functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

**Context**

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L160
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L164
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L52
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L87
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L111
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L156
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L161
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L136
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L19

## [G-06] Internal function is not called and can be removed

The function below is not called anywhere across the codebase and can be removed.

**Context**

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/VRFv2RNSource.sol#L32