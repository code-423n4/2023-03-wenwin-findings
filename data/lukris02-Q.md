# QA Report for Wenwin contest
## Overview
During the audit, 9 low and 5 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
L-1 | Change ticket type from uint256 to uint120 | Low | 2
L-2 | If  ```refferer``` == address (0), make ```refferer``` = msg.sender | Low | 1 
L-3 | Misleading comment | Low | 1
L-4 | Validate ```nonJackpotFixedRewards``` | Low | 1
L-5 | Use SafeCast Library | Low | 8
L-6 | Validate minimum values | Low | 2
L-7 | Change constant names | Low | 2
L-8 | Make contracts more universal | Low | 3
L-9 | Consider returning zero value instead of reverting | Low | 1
NC-1 | Order of Functions | Non-Critical | 5
NC-2 | Order of Layout | Non-Critical | 2
NC-3 | Prevent zero transfers | Non-Critical | 5
NC-4 | Unused named return variables | Non-Critical | 8
NC-5 | Missing leading underscores  | Non-Critical | 15

## Low Risk Findings(9)
### L-1. Change ticket type from uint256 to uint120
##### Description
In the modifier [```requireValidTicket(uint256 ticket)```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L44) and in the function [```isValidTicket()```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L18), ticket has uint256 type, though in the [```registerTicket()```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L183) function, ticket has uint128 type. And even [here](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L6) and [here](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L13) it is written that "Ticket is represented as uint120 packed ticket".

##### Instances
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L44
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L18
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L189

##### Recommendation
In the modifier [```requireValidTicket(uint256 ticket)```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L44) and in the function [```isValidTicket()```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L18) change ticket type to uint120.
#
### L-2. If  ```refferer`` == address (0), make ```refferer`` = msg.sender
##### Description
The protocol wants to maximize efficiency for participants, so maybe if the [referrer](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L114) is set to address(0) in the buyTickets() function, then make msg.sender the referrer. The user could buy a lot of tickets (and pass refferal requirements) but find out the opportunity to set themself a refferer lately - it will be sad to know about missed “bonus” ).
##### Instances
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L114

##### Recommendation
```
if  (refferer == address (0)) {
    refferer = msg.sender;
}
```
#
### L-3. Misleading comment
##### Description
The comment is "0 - staking reward, 1 - frontend reward", but it should be  "0 - frontend reward, 1 - staking reward".
##### Instances
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol#L72

##### Recommendation
Change the comment.
#
### L-4. Validate ```nonJackpotFixedRewards```
##### Description
It can be validated that rewards are in ascending order, for example: reward for (3/7) < reward for (4/7) < reward for (5/7) and so on.
##### Instances
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L91

#
### L-5. Use SafeCast Library
##### Description
Downcasting from uint256/int256 in Solidity does not revert on overflow. This can easily result in undesired exploitation or bugs, since developers usually assume that overflows raise errors. SafeCast restores this intuition by reverting the transaction when such an operation overflows.

##### Instances
Although it is impossible that somebody will buy so many tickets ( >= uint128.max + 1), it is still better to do not use unsafe type casting to avoid even theoretical truncation and prevent loss of user funds:
- [```unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L69)
- [```unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L71)

Other instances:
- [```uint16 reward = uint16(rewards[winTier] / divisor);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L170) 
- [```currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275) 
- [```newProfit = oldProfit + int256(ticketsSalesToPot);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L48)
- [```newProfit -= int256(expectedRewardsOut);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L55) 
- [```excessPotInt -= int256(fixedJackpotSize);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L64) 
- [```excessPot = excessPotInt > 0 ? uint256(excessPotInt) : 0;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L65) 

##### Recommendation
It is better to use [safe casting library](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast).
#
### L-6. Validate minimum values
##### Description
It can be validated that ```MAX_MAX_FAILED_ATTEMPTS``` and ```MAX_REQUEST_DELAY``` are not set to too small values.
##### Instances
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L27
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L30

##### Recommendation
Change to:
```
        if (_maxFailedAttempts > MAX_MAX_FAILED_ATTEMPTS || _maxFailedAttempts < MIN_MAX_FAILED_ATTEMPTS) {
            revert MaxFailedAttemptsTooBigOrTooSmall();
        }
        if (_maxRequestDelay > MAX_REQUEST_DELAY || _maxRequestDelay < MIN_REQUEST_DELAY) {
            revert MaxRequestDelayTooBigOrTooSmall();
```
#
### L-7. Change constant names
##### Description
[Here](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L19-L20) one constant has double "MAX" but other - single:
```
    uint256 private constant MAX_MAX_FAILED_ATTEMPTS = 10;
    uint256 private constant MAX_REQUEST_DELAY = 5 hours;
```
##### Instances
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L19-L20 

##### Recommendation
Change to:
```
    uint256 private constant MAX_FAILED_ATTEMPTS = 10;
    uint256 private constant MAX_REQUEST_DELAY = 5 hours;
```
or
```
    uint256 private constant MAX_MAX_FAILED_ATTEMPTS = 10;
    uint256 private constant MAX_MAX_REQUEST_DELAY = 5 hours;
```

#
### L-8. Make contracts more universal
##### Description
Protocol team wants to make different lotteries with different parameters without changing contracts, so it is better to change some constants to immutables and add parameters.
##### Instances
- [```uint256 public constant override INITIAL_SUPPLY = 1_000_000_000e18;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L12)  - change to immutable and set in constructor
- [```constructor() ERC20("Wenwin Lottery", "LOT") {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L17) - ```name_``` and ```symbol_``` should be parameters
- [```nativeToken = new LotteryToken();```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L78) - add the parameters

#
### L-9. Consider returning zero value instead of reverting
##### Description
Consider returning zero value instead of reverting in the [function claimWinningTicket](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L263). So, the user can pass as parameter (in the [function claimWinningTickets](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L170)) all their ticketIds — user just will receive rewards  without checking which tickets are winning and which are not.
##### Instances
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L263

##### Recommendation
Change:
```
revert NothingToClaim(ticketId);
```
to:
```
return 0;
```
#

## Non-Critical Risk Findings(5)
### NC-1. Order of Functions
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-functions), ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.  
Functions should be grouped according to their visibility and ordered:
1) constructor
2) receive function (if exists)
3) fallback function (if exists)
4) external
5) public
6) internal
7) private
##### Instances
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L103-L106 external functions should be placed before public
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L234-L236 public functions should be placed before internal
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120-L130 public functions should be placed after external
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L181-L196 private functions should be placed after internal
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L238-L277 private functions should be placed after internal

##### Recommendation
Reorder functions where possible.
#
### NC-2. Order of Layout
##### Description
According to [Order of Layout](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout), inside each contract, library or interface, use the following order:
1) Type declarations
2) State variables
3) Events
4) Modifiers
5) Functions
##### Instances
Modifiers should be placed before constructor:
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L104-L118 

Events should be placed before functions:
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol#L61-L71

##### Recommendation
Place events and modifiers before functions.
#
### NC-3. Prevent zero transfers
##### Description
It can be checked that amount/claimedAmount/tickets.length > 0 to avoid zero transfers and do not spend gas.
##### Instances
- [```stakedToken.transferFrom(msg.sender, address(this), amount);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L34)
- [```stakedToken.transfer(msg.sender, amount);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L47)
- [```rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L130) 
- [```rewardToken.safeTransfer(beneficiary, claimedAmount);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L156) 
- [```rewardToken.safeTransfer(msg.sender, claimedAmount);```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L175) 

##### Recommendation
Before the transfer check that transferredAmount > 0.
#
### NC-4. Unused named return variables
##### Description
Both named return variable(s) and return statement are used.
##### Instances
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L62
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L235
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L239
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L158
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L162
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L18
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L23
- https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L32

##### Recommendation
To improve clarity use only named return variables.  
For example, change:
```
function functionName() returns (uint id) {
    return x;
```
to
```
function functionName() returns (uint id) {
    id = x;
```
#
### NC-5. Missing leading underscores
##### Description
Internal and private constants, immutables, state variables and functions should have a leading underscore.
##### Instances
Constants:
- [```uint256 private constant BASE_JACKPOT_PERCENTAGE = 30_030; // 30.03%```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L36) 

Immutables:
- [```uint256 internal immutable firstDrawSchedule;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L26) 
- [```uint256 private immutable nonJackpotFixedRewards;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L34)

State variables:
- [```uint256 private claimedStakingRewardAtTicketId;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L25)
- [```mapping(address => uint256) private frontendDueTicketSales;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L26)
- [```mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L27)

Functions:
- [```function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L164)
- [```function registerTicket(```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L181)
- [```function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L203)
- [```function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L238)
- [```function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L249)
- [```function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L259)
- [```function returnUnclaimedJackpotToThePot() private {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L271)
- [```function requireFinishedDraw(uint128 drawId) internal view override {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L279)
- [```function mintNativeTokens(address mintTo, uint256 amount) internal override {```](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L285)

##### Recommendation
Add leading underscores where needed.