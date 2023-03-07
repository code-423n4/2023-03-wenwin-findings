
### Gas Optimizations List
// in addition to https://gist.github.com/Picodes/70e749bf5bbebbd5cc745ba7bc3826b4#GAS-8
| Number | Optimization Details                                                                 | Context |
|:------:| :----------------------------------------------------------------------------------- |:-------:|
| [G-01] | Setting the _constructor_ to `payable`                                               |   10    |
| [G-02] | Use constants instead of type(uint16).max                                            |    1    |
| [G-03] | Remove redundant variables                                                           |    2    |
| [G-04] | Cache array length                                                                   |    2    |
| [G-05] | Use nested if and avoid multiple check combinations                                  |    2    |
| [G-06] | Functions guaranteed to revert\_ when called by normal users can be marked `payable` |    4    |
| [G-07] | Sort Solidity operations using short-circuit mode                                    |    2    |
| [G-08] | `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables    |    8    |
| [G-09] | Optimize names to save gas                                                           |   All   |
| [G-10] | Multiple accesses of a mapping/array should use a local variable cache               |    7    |
| [G-11] | remove `else if`                                                                     |    1    |
| [G-12] |A modifier used only once and not being inherited should be inlined to save gas|    6    |

Total 45 issues

### [G-01] Setting the _constructor_ to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable.

**Context:**
10 results - 10 files:

```solidity
src/RNSourceBase.sol:11
    constructor(address _authorizedConsumer) {
```

[RNSourceBase.sol#L11](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceBase.sol#L11)

```solidity
src/Lottery.sol:84
    constructor(
```

[Lottery.sol#L84](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L84)

```solidity
src/LotterySetup.sol:41
    constructor(LotterySetupParams memory lotterySetupParams) {
```

[LotterySetup.sol#L41](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L41)

```solidity
src/LotteryToken.sol:17
    constructor() ERC20("Wenwin Lottery", "LOT") {
```

[LotteryToken.sol#L17](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotteryToken.sol#L17)

```solidity
src/RNSourceBase.sol:11
    constructor(address _authorizedConsumer) {
```

[RNSourceBase.sol#L11](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceBase.sol#L11)

```solidity
src/RNSourceController.sol:26
    constructor(address _authorizedConsumer) {
```

[RNSourceController.sol#L26](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceController.sol#L26)

```solidity
src/ReferralSystem.sol:27
    constructor(
```

[src/ReferralSystem.sol:27](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L27)

```solidity
src/VRFv2RNSource.sol:13
    constructor(
```

[src/VRFv2RNSource.sol:13](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/VRFv2RNSource.sol#L13)

```solidity
src/staking/StakedTokenLock.sol:16
    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
```

[src/staking/StakedTokenLock.sol:16](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/StakedTokenLock.sol#L16)

```solidity
src/staking/Staking.sol:22
    constructor(
```

[src/staking/Staking.sol:22](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/Staking.sol#L22)

### [G-02] Use constants instead of type(uint16).max

type(uint16).max etc. uses more gas in the distribution process and also for each transaction than constant usage. We can
remove uint256 cast as well.

**Context:**
1 results 1 files:

```diff
src/LotterySetup.sol:126
- :126            uint256 mask = uint256(type(uint16).max) << (winTier * 16);
+ :126            uint256 mask = 65_535 << (winTier << 4);
```

[src/LotterySetup.sol:126](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L126)

### [G-04] Remove redundant variables

We can safely remove `raised` variable and `assert` function as it no longer needed.
`initialPot` will be reverted if transaction fails. We check that `initialPot > minInitialPot` thus it will be more than 0;

**Context:**
2 results 2 files:

```solidity
src/LotterySetup.sol:132
    function finalizeInitialPotRaise() external override {
        if (initialPot > 0) {
            revert JackpotAlreadyInitialized();
        }
        // slither-disable-next-line timestamp
        if (block.timestamp <= initialPotDeadline) {
            revert FinalizingInitialPotBeforeDeadline();
        }
        uint256 raised = rewardToken.balanceOf(address(this));
        if (raised < minInitialPot) {
            revert RaisedInsufficientFunds(raised);
        }
        initialPot = raised;

        // must hold after this call, this will be used as a check that jackpot is initialized
        assert(initialPot > 0);

        emit InitialPotPeriodFinalized(raised);
    }
```

Recommendation Code:

```solidity
src/LotterySetup.sol:132
    function finalizeInitialPotRaise() external override {
        if (initialPot > 0) revert JackpotAlreadyInitialized();
        // slither-disable-next-line timestamp
        if (block.timestamp <= initialPotDeadline) revert FinalizingInitialPotBeforeDeadline();

        initialPot = rewardToken.balanceOf(address(this));
        if (initialPot < minInitialPot) revert RaisedInsufficientFunds(initialPot);

        emit InitialPotPeriodFinalized(initialPot);
    }
```

[src/LotterySetup.sol:132](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L132)

Here we have two variables that can be removed. They were introduced for readability. Maybe you will consider removing them

```solidity
src/RNSourceController.sol:93
        bool notEnoughRetryInvocations = failedSequentialAttempts < maxFailedAttempts;
        bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;
        if (notEnoughRetryInvocations || notEnoughTimeReachingMaxFailedAttempts) {
            revert NotEnoughFailedAttempts();
        }
```

Recommendation Code:

```solidity
src/RNSourceController.sol:93
        if (
            failedSequentialAttempts < maxFailedAttempts
                || block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay
        ) revert NotEnoughFailedAttempts();
```

[src/RNSourceController.sol:93](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceController.sol#L93)

### [G-04] Cache array length

If not cached, the solidity compiler will always read the length of the array during every time.
`tickets.length`, `drawIds.length`

**Context:** 2 results - 1 files:

```solidity
src/LotterySetup.sol:126
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
        for (uint256 i = 0; i < drawIds.length; ++i) {
            ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
        }
        referralRegisterTickets(currentDraw, referrer, msg.sender, tickets.length);
        frontendDueTicketSales[frontend] += tickets.length;
        rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);
    }
```

### [G-05] Use nested if and avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

**Context:**
2 results - 2 files:

```solidity
src/LotteryMath.sol:83
        if (excessPot > 0 && ticketsSold > 0) {
```

[src/LotteryMath.sol:83](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotteryMath.sol#L83)

```solidity
src/staking/StakedTokenLock.sol:39
        if (excessPot > 0 && ticketsSold > 0) {
```

[src/staking/StakedTokenLock.sol:39](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/StakedTokenLock.sol#L39)
**Recomendation Code:**

```diff
src/LotteryMath.sol:83
-        if (excessPot > 0 && ticketsSold > 0) {
+        if (excessPot > 0) {
+           if (ticketsSold > 0) {
            bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
        }
```

### [G-06] Functions guaranteed to revert\_ when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

**Context:**
4 results - 2 files:

```solidity
src/staking/StakedTokenLock.sol
24:     function deposit(uint256 amount) external override onlyOwner {
37:     function withdraw(uint256 amount) external override onlyOwner {
```

[src/staking/StakedTokenLock.sol:24](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/StakedTokenLock.sol#L24)

```solidity
src/RNSourceController.sol
77:     function initSource(IRNSource rnSource) external override onlyOwner {
89:     function swapSource(IRNSource newSource) external override onlyOwner {
```

[RNSourceController.sol#L77](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceController.sol#L77)

### [G-07] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses `OR/AND` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation
//g(y) is a high gas cost operation

//Sort operations with different gas costs as follows
f(x) || g(y)
f(x) && g(y)
```

**Context:**
2 results - 1 files:

```solidity
src/LotterySetup.sol:54
54:        if (
55:            lotterySetupParams.expectedPayout < lotterySetupParams.ticketPrice / 100
56:               || lotterySetupParams.expectedPayout >= lotterySetupParams.ticketPrice
57:        ) {

165:                if (rewards.length != (selectionSize) || rewards[0] != 0) {
```

[src/LotterySetup.sol:54](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L54)

### [G-08] `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables

**Context:**
8 results - 3 files:

```solidity
src/Lottery.sol
129:        frontendDueTicketSales[frontend] += tickets.length;

275:        currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

[src/Lottery.sol:129](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L129)

```solidity
src/ReferralSystem.sol
64:            totalTicketsForReferrersPerDraw[currentDraw] +=

67:                            totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;

69:            unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

71:        unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
```

[src/ReferralSystem.sol:64](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L64)

```solidity
src/staking/StakedTokenLock.sol
30:     depositedBalance += amount;
43:     depositedBalance -= amount;
```

[src/staking/StakedTokenLock.sol:30](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/StakedTokenLock.sol#L30)

### [G-09] Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via `Method ID`. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because `22 gas` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

**Context:**
All Contracts

**Recommendation:**
Find a lower `method ID` name for the most called functions for example Call() vs. Call1() is cheaper by `22 gas`
For example, the function IDs in the `SmartAccount.sol` contract will be the most used; A lower method ID may be given.

**Proof of Consept:**
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

### [G-10] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed multiple times, saves gas

**Context:**
7 results - 5 files:

```solidity
src/ReferralSystem.sol:136
    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
        requireFinishedDraw(drawId);

        UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {
            claimedReward = referrerRewardPerDrawForOneTicket[drawId] * _unclaimedTickets.referrerTicketCount;
            unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
        }

        _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.playerTicketCount > 0) {
            claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
            unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
        }

        if (claimedReward > 0) {
            emit ClaimedReferralReward(drawId, msg.sender, claimedReward);
        }
    }
```

Recommendation Code:

```solidity
src/ReferralSystem.sol:136
    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
        requireFinishedDraw(drawId);

        UnclaimedTicketsData storage _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {
            claimedReward = referrerRewardPerDrawForOneTicket[drawId] * _unclaimedTickets.referrerTicketCount;
            _unclaimedTickets.referrerTicketCount = 0;
        }

        if (_unclaimedTickets.playerTicketCount > 0) {
            claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
            _unclaimedTickets.playerTicketCount = 0;
        }

        if (claimedReward > 0) {
            emit ClaimedReferralReward(drawId, msg.sender, claimedReward);
        }
    }
```

[src/ReferralSystem.sol:136](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L136)

```diff
src/RNSourceBase.sol:34
-34:    if (requests[requestId].status == RequestStatus.None) {
+       RandomnessRequest storage currentRequest = requests[requestId];
+        if (currentRequest.status == RequestStatus.None) {
```

[src/RNSourceBase.sol:34](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L136)

```solidity
src/ReferralSystem.sol
62:     if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {

64:     totalTicketsForReferrersPerDraw[currentDraw] +=
```

[src/ReferralSystem.sol:62](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L62)

```solidity
src/ReferralSystem.sol
62:     if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {

64:     totalTicketsForReferrersPerDraw[currentDraw] +=
```

[src/ReferralSystem.sol:62](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L62)

```solidity
src/RNSourceBase.sol:34
if (requests[requestId].status == RequestStatus.None) {
```

[src/RNSourceBase.sol:34](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceBase.sol#L34)

### [G-11] remove `else if`

Improves test testCalculateNonJackpotReward gas, plus better readability.

**Context:**
1 results - 1 files:

```solidity
src/LotterySetup.sol:120
    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
        if (winTier == selectionSize) {
            return _baseJackpot(initialPot);
        } else if (winTier == 0 || winTier > selectionSize) {
            return 0;
        } else {
            uint256 mask = uint256(type(uint16).max) << (winTier * 16);
            uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
            return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
        }
    }
```

Recommendation Code:

```solidity
src/LotterySetup.sol:120
    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
        if (winTier == selectionSize) return _baseJackpot(initialPot);
        if (winTier == 0 || winTier > selectionSize) return 0;

        uint256 mask = uint256(type(uint16).max) << (winTier * 16);
        uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
        return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
    }
```

[src/LotterySetup.sol:120](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L120)

### [G-12] A modifier used only once and not being inherited should be inlined to save gas

**Context:**
6 results - 2 files:

```solidity
src/Lottery.sol:44
    modifier requireValidTicket(uint256 ticket) {
        if (!ticket.isValidTicket(selectionSize, selectionMax)) {
            revert InvalidTicket();
        }
        _;
    }
```

[src/Lottery.sol:44](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L44)

Only used in one place, here is recommendation

```diff
src/Lottery.sol:172
    function registerTicket(
        uint128 drawId,
        uint120 ticket,
        address frontend,
        address referrer
    )
        private
        beforeTicketRegistrationDeadline(drawId)
-       requireValidTicket(ticket)
        returns (uint256 ticketId)
    {
+        if (!uint256(ticket).isValidTicket(selectionSize, selectionMax)) revert InvalidTicket();
        ticketId = mint(msg.sender, drawId, ticket);
        unclaimedCount[drawId][ticket]++;
        ticketsSold[drawId]++;
        emit NewTicket(currentDraw, ticketId, drawId, msg.sender, ticket, frontend, referrer);
    }
```

[src/Lottery.sol:172](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L172)

Same gas optimisation:

```solidity
src/Lottery.sol
52:    modifier whenNotExecutingDraw() {

60:    modifier onlyWhenExecutingDraw() {

69:    modifier onlyTicketOwner(uint256 ticketId) {

```

[src/Lottery.sol:44](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L44)

```solidity
src/LotterySetup.sol:104
104:    modifier requireJackpotInitialized() {

112:    modifier beforeTicketRegistrationDeadline(uint128 drawId) {
```

[src/LotterySetup.sol:104](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L104)

| Number | Optimization Details                                                                 | Context |
| :----: | :----------------------------------------------------------------------------------- | :-----: |
| [G-01] | Setting the *constructor* to `payable`                                               |    10   |
| [G-02] | Use constants instead of type(uint16).max                                            |    1    |
| [G-03] | Remove redundant variables                                                           |    2    |
| [G-04] | Cache array length                                                                   |    2    |
| [G-05] | Use nested if and avoid multiple check combinations                                  |    2    |
| [G-06] | Functions guaranteed to revert\_ when called by normal users can be marked `payable` |    4    |
| [G-07] | Sort Solidity operations using short-circuit mode                                    |    2    |
| [G-08] | `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables    |    8    |
| [G-09] | Optimize names to save gas                                                           |   All   |
| [G-10] | Multiple accesses of a mapping/array should use a local variable cache               |    7    |
| [G-11] | remove `else if`                                                                     |    1    |
| [G-12] | A modifier used only once and not being inherited should be inlined to save gas      |    6    |

Total 27 issues

### [G-01] Setting the *constructor* to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable.

**Context:**
10 results - 10 files:

```solidity
src/RNSourceBase.sol:11
    constructor(address _authorizedConsumer) {
```

[RNSourceBase.sol#L11](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceBase.sol#L11)

```solidity
src/Lottery.sol:84
    constructor(
```

[Lottery.sol#L84](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L84)

```solidity
src/LotterySetup.sol:41
    constructor(LotterySetupParams memory lotterySetupParams) {
```

[LotterySetup.sol#L41](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L41)

```solidity
src/LotteryToken.sol:17
    constructor() ERC20("Wenwin Lottery", "LOT") {
```

[LotteryToken.sol#L17](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotteryToken.sol#L17)

```solidity
src/RNSourceBase.sol:11
    constructor(address _authorizedConsumer) {
```

[RNSourceBase.sol#L11](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceBase.sol#L11)

```solidity
src/RNSourceController.sol:26
    constructor(address _authorizedConsumer) {
```

[RNSourceController.sol#L26](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceController.sol#L26)

```solidity
src/ReferralSystem.sol:27
    constructor(
```

[src/ReferralSystem.sol:27](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L27)

```solidity
src/VRFv2RNSource.sol:13
    constructor(
```

[src/VRFv2RNSource.sol:13](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/VRFv2RNSource.sol#L13)

```solidity
src/staking/StakedTokenLock.sol:16
    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
```

[src/staking/StakedTokenLock.sol:16](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/StakedTokenLock.sol#L16)

```solidity
src/staking/Staking.sol:22
    constructor(
```

[src/staking/Staking.sol:22](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/Staking.sol#L22)

### [G-02] Use constants instead of type(uint16).max

type(uint16).max etc. uses more gas in the distribution process and also for each transaction than constant usage. We can
remove uint256 cast as well.

**Context:**
1 results 1 files:

```diff
src/LotterySetup.sol:126
- :126            uint256 mask = uint256(type(uint16).max) << (winTier * 16);
+ :126            uint256 mask = 65_535 << (winTier << 4);
```

[src/LotterySetup.sol:126](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L126)

### [G-04] Remove redundant variables

We can safely remove `raised` variable and `assert` function as it no longer needed.
`initialPot` will be reverted if transaction fails. We check that `initialPot > minInitialPot` thus it will be more than 0;

**Context:**
2 results 2 files:

```solidity
src/LotterySetup.sol:132
    function finalizeInitialPotRaise() external override {
        if (initialPot > 0) {
            revert JackpotAlreadyInitialized();
        }
        // slither-disable-next-line timestamp
        if (block.timestamp <= initialPotDeadline) {
            revert FinalizingInitialPotBeforeDeadline();
        }
        uint256 raised = rewardToken.balanceOf(address(this));
        if (raised < minInitialPot) {
            revert RaisedInsufficientFunds(raised);
        }
        initialPot = raised;

        // must hold after this call, this will be used as a check that jackpot is initialized
        assert(initialPot > 0);

        emit InitialPotPeriodFinalized(raised);
    }
```

Recommendation Code:

```solidity
src/LotterySetup.sol:132
    function finalizeInitialPotRaise() external override {
        if (initialPot > 0) revert JackpotAlreadyInitialized();
        // slither-disable-next-line timestamp
        if (block.timestamp <= initialPotDeadline) revert FinalizingInitialPotBeforeDeadline();

        initialPot = rewardToken.balanceOf(address(this));
        if (initialPot < minInitialPot) revert RaisedInsufficientFunds(initialPot);

        emit InitialPotPeriodFinalized(initialPot);
    }
```

[src/LotterySetup.sol:132](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L132)

Here we have two variables that can be removed. They were introduced for readability. Maybe you will consider removing them

```solidity
src/RNSourceController.sol:93
        bool notEnoughRetryInvocations = failedSequentialAttempts < maxFailedAttempts;
        bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;
        if (notEnoughRetryInvocations || notEnoughTimeReachingMaxFailedAttempts) {
            revert NotEnoughFailedAttempts();
        }
```

Recommendation Code:

```solidity
src/RNSourceController.sol:93
        if (
            failedSequentialAttempts < maxFailedAttempts
                || block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay
        ) revert NotEnoughFailedAttempts();
```

[src/RNSourceController.sol:93](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceController.sol#L93)

### [G-04] Cache array length

If not cached, the solidity compiler will always read the length of the array during every time.
`tickets.length`, `drawIds.length`

**Context:** 2 results - 1 files:

```solidity
src/LotterySetup.sol:126
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
        for (uint256 i = 0; i < drawIds.length; ++i) {
            ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
        }
        referralRegisterTickets(currentDraw, referrer, msg.sender, tickets.length);
        frontendDueTicketSales[frontend] += tickets.length;
        rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);
    }
```

### [G-05] Use nested if and avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

**Context:**
2 results - 2 files:

```solidity
src/LotteryMath.sol:83
        if (excessPot > 0 && ticketsSold > 0) {
```

[src/LotteryMath.sol:83](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotteryMath.sol#L83)

```solidity
src/staking/StakedTokenLock.sol:39
        if (excessPot > 0 && ticketsSold > 0) {
```

[src/staking/StakedTokenLock.sol:39](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/StakedTokenLock.sol#L39)
**Recomendation Code:**

```diff
src/LotteryMath.sol:83
-        if (excessPot > 0 && ticketsSold > 0) {
+        if (excessPot > 0) {
+           if (ticketsSold > 0) {
            bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
        }
```

### [G-06] Functions guaranteed to revert\_ when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

**Context:**
4 results - 2 files:

```solidity
src/staking/StakedTokenLock.sol
24:     function deposit(uint256 amount) external override onlyOwner {
37:     function withdraw(uint256 amount) external override onlyOwner {
```

[src/staking/StakedTokenLock.sol:24](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/StakedTokenLock.sol#L24)

```solidity
src/RNSourceController.sol
77:     function initSource(IRNSource rnSource) external override onlyOwner {
89:     function swapSource(IRNSource newSource) external override onlyOwner {
```

[RNSourceController.sol#L77](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceController.sol#L77)

### [G-07] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses `OR/AND` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

    //f(x) is a low gas cost operation
    //g(y) is a high gas cost operation

    //Sort operations with different gas costs as follows
    f(x) || g(y)
    f(x) && g(y)

**Context:**
2 results - 1 files:

```solidity
src/LotterySetup.sol:54
54:        if (
55:            lotterySetupParams.expectedPayout < lotterySetupParams.ticketPrice / 100
56:               || lotterySetupParams.expectedPayout >= lotterySetupParams.ticketPrice
57:        ) {

165:                if (rewards.length != (selectionSize) || rewards[0] != 0) {
```

[src/LotterySetup.sol:54](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L54)

### [G-08] `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables

**Context:**
8 results - 3 files:

```solidity
src/Lottery.sol
129:        frontendDueTicketSales[frontend] += tickets.length;

275:        currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

[src/Lottery.sol:129](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L129)

```solidity
src/ReferralSystem.sol
64:            totalTicketsForReferrersPerDraw[currentDraw] +=

67:                            totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;

69:            unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

71:        unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
```

[src/ReferralSystem.sol:64](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L64)

```solidity
src/staking/StakedTokenLock.sol
30:     depositedBalance += amount;
43:     depositedBalance -= amount;
```

[src/staking/StakedTokenLock.sol:30](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/StakedTokenLock.sol#L30)

### [G-09] Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via `Method ID`. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because `22 gas` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

**Context:**
All Contracts

**Recommendation:**
Find a lower `method ID` name for the most called functions for example Call() vs. Call1() is cheaper by `22 gas`
For example, the function IDs in the `SmartAccount.sol` contract will be the most used; A lower method ID may be given.

**Proof of Consept:**


### [G-10] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed multiple times, saves gas

**Context:**
7 results - 5 files:

```solidity
src/ReferralSystem.sol:136
    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
        requireFinishedDraw(drawId);

        UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {
            claimedReward = referrerRewardPerDrawForOneTicket[drawId] * _unclaimedTickets.referrerTicketCount;
            unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
        }

        _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.playerTicketCount > 0) {
            claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
            unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
        }

        if (claimedReward > 0) {
            emit ClaimedReferralReward(drawId, msg.sender, claimedReward);
        }
    }
```

Recommendation Code:

```solidity
src/ReferralSystem.sol:136
    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
        requireFinishedDraw(drawId);

        UnclaimedTicketsData storage _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {
            claimedReward = referrerRewardPerDrawForOneTicket[drawId] * _unclaimedTickets.referrerTicketCount;
            _unclaimedTickets.referrerTicketCount = 0;
        }

        if (_unclaimedTickets.playerTicketCount > 0) {
            claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;
            _unclaimedTickets.playerTicketCount = 0;
        }

        if (claimedReward > 0) {
            emit ClaimedReferralReward(drawId, msg.sender, claimedReward);
        }
    }
```

[src/ReferralSystem.sol:136](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L136)

```diff
src/RNSourceBase.sol:34
-34:    if (requests[requestId].status == RequestStatus.None) {
+       RandomnessRequest storage currentRequest = requests[requestId];
+        if (currentRequest.status == RequestStatus.None) {
```

[src/RNSourceBase.sol:34](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L136)

```solidity
src/ReferralSystem.sol
62:     if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {

64:     totalTicketsForReferrersPerDraw[currentDraw] +=
```

[src/ReferralSystem.sol:62](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L62)

```solidity
src/ReferralSystem.sol
62:     if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {

64:     totalTicketsForReferrersPerDraw[currentDraw] +=
```

[src/ReferralSystem.sol:62](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L62)

```solidity
src/RNSourceBase.sol:34
if (requests[requestId].status == RequestStatus.None) {
```

[src/RNSourceBase.sol:34](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/RNSourceBase.sol#L34)

### [G-11] remove `else if`

Improves test testCalculateNonJackpotReward gas, plus better readability.

**Context:**
1 results - 1 files:

```solidity
src/LotterySetup.sol:120
function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
if (winTier == selectionSize) {
return _baseJackpot(initialPot);
} else if (winTier == 0 || winTier > selectionSize) {
return 0;
} else {
uint256 mask = uint256(type(uint16).max) << (winTier * 16);
uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
}
}
```

Recommendation Code:

```solidity
src/LotterySetup.sol:120
function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
if (winTier == selectionSize) return _baseJackpot(initialPot);
if (winTier == 0 || winTier > selectionSize) return 0;

uint256 mask = uint256(type(uint16).max) << (winTier * 16);
uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
}
```

[src/LotterySetup.sol:120](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L120)

### [G-12] A modifier used only once and not being inherited should be inlined to save gas

**Context:**
6 results - 2 files:

```solidity
src/Lottery.sol:44
modifier requireValidTicket(uint256 ticket) {
if (!ticket.isValidTicket(selectionSize, selectionMax)) {
revert InvalidTicket();
}
_;
}
```

[src/Lottery.sol:44](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L44)

Only used in one place, here is recommendation

```diff
src/Lottery.sol:172
    function registerTicket(
        uint128 drawId,
        uint120 ticket,
        address frontend,
        address referrer
    )
        private
        beforeTicketRegistrationDeadline(drawId)
-       requireValidTicket(ticket)
        returns (uint256 ticketId)
    {
+        if (!uint256(ticket).isValidTicket(selectionSize, selectionMax)) revert InvalidTicket();
        ticketId = mint(msg.sender, drawId, ticket);
        unclaimedCount[drawId][ticket]++;
        ticketsSold[drawId]++;
        emit NewTicket(currentDraw, ticketId, drawId, msg.sender, ticket, frontend, referrer);
    }
```

[src/Lottery.sol:172](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L172)

Same gas optimisation:

```solidity
src/Lottery.sol
52:    modifier whenNotExecutingDraw() {

60:    modifier onlyWhenExecutingDraw() {

69:    modifier onlyTicketOwner(uint256 ticketId) {
```

[src/Lottery.sol:44](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L44)

```solidity
src/LotterySetup.sol:104
104:    modifier requireJackpotInitialized() {

112:    modifier beforeTicketRegistrationDeadline(uint128 drawId) {
```

[src/LotterySetup.sol:104](https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotterySetup.sol#L104)