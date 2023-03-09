### [G-01] Functions guaranteed to revert when called by normal users can be declared as payable.


#### Impact
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.


#### Findings:
```
./RNSourceController.sol:L77    function initSource(IRNSource rnSource) external override onlyOwner {

./RNSourceController.sol:L89    function swapSource(IRNSource newSource) external override onlyOwner {

./Lottery.sol:L203    function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {

./Lottery.sol:L259    function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {

./staking/StakedTokenLock.sol:L24    function deposit(uint256 amount) external override onlyOwner {

./staking/StakedTokenLock.sol:L37    function withdraw(uint256 amount) external override onlyOwner {

```

### [G-02] ```X += Y``` costs more gas than ```X = X + Y``` for state variables.


#### Impact
Consider changing ```X += Y``` to ```X = X + Y``` to save gas.


#### Findings:
```
./LotteryMath.sol:L55        newProfit -= int256(expectedRewardsOut);

./LotteryMath.sol:L64        excessPotInt -= int256(fixedJackpotSize);

./LotteryMath.sol:L84            bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);

./TicketUtils.sol:L29                ticketSize += (ticket & uint256(1));

./TicketUtils.sol:L96                winTier += uint8(intersection & uint120(1));

./Lottery.sol:L129        frontendDueTicketSales[frontend] += tickets.length;

./Lottery.sol:L173            claimedAmount += claimWinningTicket(ticketIds[i]);

./Lottery.sol:L275            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);

./ReferralSystem.sol:L67                totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;

./ReferralSystem.sol:L69            unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

./ReferralSystem.sol:L71        unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);

./ReferralSystem.sol:L78            claimedReward += claimPerDraw(drawIds[counter]);

./ReferralSystem.sol:L147            claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;

./staking/StakedTokenLock.sol:L30        depositedBalance += amount;

./staking/StakedTokenLock.sol:L43        depositedBalance -= amount;

```
