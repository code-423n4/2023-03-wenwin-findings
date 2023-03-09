## Low and Non-Critical Issues Summary
| Number |Issues Details |
|:--:|:-------|
|[L-01]| MISSING EVENT FOR CRITICAL PARAMETER CHANGE
|[NC-01]| Use latest Solidity version
|[NC-02]| Use stable pragma statement
|[NC-03]| Different pragma directives are used
|[NC-04]| Use named imports instead of plain `import FILE.SOL`
|[NC-05]| In all solidity files, license keyword should be mentioned
|[NC-06]| Constants should be defined rather than using magic numbers
|[NC-07]| Add NatSpec documentation
|[NC-08]| You can use named parameters in mapping types
***
## [L-01] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

Emitting events allows monitoring activities with off-chain monitoring tools.

```
Lottery.sol
110: function buyTickets(
170: function claimWinningTickets(uint256[] calldata ticketIds) external override returns (uint256 claimedAmount) {
```
***
## [NC-01] Use latest Solidity version

Solidity pragma versioning should be upgraded to latest available version. 

```
VRFv2RNSource.sol
pragma solidity ^0.8.7;
```
***

## [NC-02] Use stable pragma statement

Using a floating pragma statement `^0.8.7` is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.

***
## [NC-03] Different pragma directives are used

Use one Solidity version on each contract.
***

## [NC-04] USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT FILE.SOL`

**Recommendation:**
`import {contract1 , contract2} from "filename.sol";
***

## [NC-05] In all solidity files, license keyword should be mentioned
***

## [NC-06] Constants should be defined rather than using magic numbers

```solidity
LotterySetup.sol
36: uint256 private constant BASE_JACKPOT_PERCENTAGE = 30_030
51: if (lotterySetupParams.selectionMax >= 120) {
55: lotterySetupParams.expectedPayout < lotterySetupParams.ticketPrice / 100
61: lotterySetupParams.selectionSize > 16
81: jackpotBound = 2_000_000 * tokenUnit;
126: uint256 mask = uint256(type(uint16).max) << (winTier * 16);
127: uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);

ReferralSystem.sol
117: if (totalTicketsSoldPrevDraw < 10_000) {
121: if (totalTicketsSoldPrevDraw < 100_000) {
123: return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);
125: (totalTicketsSoldPrevDraw < 1_000_000) {
127: return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);
```
***
## [NC-07] Add NatSpec documentation

NatSpec documentation to all public/external functions and variables is essential for better understanding of the code by developers and auditors and is strongly recommended.
***

## [NC-08] You can use named parameters in mapping types

From Solidity [0.8.18](https://blog.soliditylang.org/2023/02/01/solidity-0.8.18-release-announcement/) you can use named parameters in mapping types. This will make the code much more readable.