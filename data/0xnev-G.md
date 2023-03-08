### Gas Optimization Template
| Count | Title | Instances |
|:--:|:-------|:--:|
| [G-01] | Unecessary initialization of named return variable | 9 |
| [G-02] | Use `delete` instead of default value assignment to clear storage variables  | 10 |
| [G-03] | Use nested `if` statements to avoid multiple check combinations using `&&` | 2 |
| [G-04] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (same with `-=`) | 8 |
| [G-05] | Multiple `address`/`uint` mappings can be combined into a single mapping of an address to a struct, where appropriate| 3 |
| [G-06] | Multiple accesses of a mapping/array should use a local variable cache | 7 |
| [G-07] | Check amount before execution of `transfer` functions for possible gas savings | 4 |
| [G-08] | `internal/private` functions only called once can be inlined to save gas | 4 |
| [G-09] | Inline a `modifer` that is only used once | 6 |
| [G-10] | Check amount before execution of functions for possible gas savings | 2 |
| [G-11] | Usage of `uint` smaller than 32 bytes (256 bits) incurs overhead | 1 |

| Total Gas-Optimization Issues | 11 |
|:--:|:--:|

### [G-01] Unecessary initialization of named return variable 
Context:

Some functions declared a named return variable but the variable is not used elsewhere:
```solidity
9 results - 6 files

/Staking.sol
 61:    function earned(address account) public view override returns (uint256 _earned) {
 62:        return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];
 63:    }

/LotterySetup.sol
120:    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
121:        if (winTier == selectionSize) {
122:            return _baseJackpot(initialPot);
123:        } else if (winTier == 0 || winTier > selectionSize) {
124:            return 0;
125:        } else {
126:            uint256 mask = uint256(type(uint16).max) << (winTier * 16);
127:            uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
128:            return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
129:        }
130:    }

/Lottery.sol
234:    function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
235:        return drawRewardSize(currentDraw, winTier);
236:    }

/ReferralSystem.sol
111:    function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
112:        internal
113:        view
114:        virtual
115:        returns (uint256 minimumEligible)
116:    {
117:        if (totalTicketsSoldPrevDraw < 10_000) {
118:            // 1%
119:            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT);
120:        }
121:        if (totalTicketsSoldPrevDraw < 100_000) {
122:            // 0.75%
123:            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);
124:        }
125:        if (totalTicketsSoldPrevDraw < 1_000_000) {
126:            // 0.5%
127:            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);
128:        }
129:        return 5000;
130    }

156:    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
157:        uint256 decrease = uint256(drawId) * playerRewardDecreasePerDraw;
158:        return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;
159:    }

161:    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
162:        return rewardsToReferrersPerDraw[Math.min(rewardsToReferrersPerDraw.length - 1, drawId)];
163:    }

/PercentageMath.sol
 17:    function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
 18:        return number * percentage / PERCENTAGE_BASE;
 19:    }

 22:    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
 23:        return number * int256(percentage) / int256(PERCENTAGE_BASE);
 24:    }

 /TicketUtils.sol
 17:    function isValidTicket(
 18:       uint256 ticket,
 19:       uint8 selectionSize,
 20:       uint8 selectionMax
 21:   )
 22:       internal
 23:       pure
 24:       returns (bool isValid)
 25:   {
 26:       unchecked {
 27:           uint256 ticketSize;
 28:           for (uint8 i = 0; i < selectionMax; ++i) {
 29:               ticketSize += (ticket & uint256(1));
 30:               ticket >>= 1;
 31:           }
 32:           return (ticketSize == selectionSize) && (ticket == uint256(0));
 33:       }
 34:   }
```
Reccomendation:
Consider returning an unnamed variable

### [G-02] Use `delete` instead of default value assignment to clear storage variables 
Context:
```solidity
10 results - 3 files

/Staking.sol
 95: rewards[msg.sender] = 0;

/Lottery.sol
225: drawExecutionInProgress = false;
255: frontendDueTicketSales[beneficiary] = 0;

/RNSourceController.sol
 52: failedSequentialAttempts = 0;
 53: maxFailedAttemptsReachedAt = 0;
 99: failedSequentialAttempts = 0;
100: maxFailedAttemptsReachedAt = 0;
108: lastRequestFulfilled = false;

/ReferralSystem.sol
142: unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
148: unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
```

Description: 
delete `a` assigns the initial value for the type to `a`. (`0` for `uint/int`, `false` for `bool`, `address(0)` for `address` etc..)

While `delete` has no effect on mappings, deleting a particular value of key in mappings is possible through recursion for nested structs and even saves gas 
https://docs.soliditylang.org/en/v0.8.19/types.html?highlight=delete#delete

Recommendation:
Use delete instead of zero assignment to clear storage variables and sdave gas

### [G-03] Use nested `if` statements to avoid multiple check combinations using `&&`
Context:
```solidity
2 results - 2 files

/StakedTokenLock.sol
39: if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration)

/LotteryMath.sol
83: if (excessPot > 0 && ticketsSold > 0)
```

Description:
Using nested `if` statements is cheaper than using `&& ` for multiple check combinations. Additionally, it improves readability of code and better coverage reports.

Recommendation:
Replace `&&` used within `if` and `else if` statements with nested `if` statements

### [G-04] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (same with `-=`)

```solidity
8 results - 3 files

/StakedTokenLock.sol
 30: depositedBalance += amount;
 43: depositedBalance -= amount;

/Lottery.sol
129: frontendDueTicketSales[frontend] += tickets.length;
275: currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);

/ReferralSystem.sol
64: totalTicketsForReferrersPerDraw[currentDraw] +=
65:     unclaimedTickets[currentDraw][referrer].referrerTicketCount;

67: totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
69: unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
71: unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
```

Recommendation:
Replace `<x> += <y>` with `<x> = <x> + <y>` for state variables (same for `-=`)

### [G-05] Multiple `address`/`uint` mappings can be combined into a single mapping of an address to a struct, where appropriate
Context:
```solidity
3 results - 3 files

/Staking.sol
19: mapping(address => uint256) public override userRewardPerTokenPaid;
20: mapping(address => uint256) public override rewards;

/Lottery.sol
27: mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
36: mapping(uint128 => uint120) public override winningTicket;
37: mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
39: mapping(uint128 => uint256) public override ticketsSold;

/ReferralSystem.sol
17: mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;
19: mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;
21: mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;
23: mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;
25: mapping(uint128 => uint256) public override minimumEligibleReferrals;
```

Description:
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot

Recommendation:
Combine mappings multiple address mappings to a single mapping of an address to a struct, where appropriate

### [G-06] Multiple accesses of a mapping/array should use a local variable cache
Description:
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory

Recommendation:
Cache variable like this
`SomeValue storage someValue = someMap[someIndex]`

Context:
```solidity
7 results - 4 files
```

#### Staking.sol
Cache `rewards[msg.sender]` in local storage

```solidity
91:    function getReward() public override
93:        uint256 reward = rewards[msg.sender];
95:            rewards[msg.sender] = 0;
```

#### Lottery.sol
Cache `frontendDueTicketSales[beneficiary]` in local storage

```solidity
249:    function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets)
254:            dueTickets = frontendDueTicketSales[beneficiary];
255:            frontendDueTicketSales[beneficiary] = 0;
```

#### RNSourceBase.sol
Cache `requests[requestId]` in local storage

```solidity
17:    function requestRandomNumber() external override
23:        if (requests[requestId].status != RequestStatus.None)
27:        requests[requestId] = RandomnessRequest({ status: RequestStatus.Pending, randomNumber: 0 });

33:    function fulfill(uint256 requestId, uint256 randomNumber) internal
34:        if (requests[requestId].status == RequestStatus.None)
37:        if (requests[requestId].status == RequestStatus.Fulfilled)
41:        requests[requestId].randomNumber = randomNumber;
42:        requests[requestId].status = RequestStatus.Fulfilled;
```

#### ReferralSystem.sol
Cache `unclaimedTickets[currentDraw][referrer]` in local storage

```solidity
52:    function referralRegisterTickets
62:            if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) 
63:                if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible)
65:                         unclaimedTickets[currentDraw][referrer].referrerTicketCount;
69:            unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
71:         unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
```

Cache `unclaimedTickets[drawId][msg.sender]` in local storage
```solidity
136:    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward)
139:        UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
142:            unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
145:        _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
148:            unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
```

Cache `totalTicketsForReferrersPerDraw[currentDraw]` in local storage

```solidity
52:    function referralRegisterTickets
64:                     totalTicketsForReferrersPerDraw[currentDraw] +=
67:                totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;
```

### [G-08] `internal/private` functions only called once can be inlined to save gas

Description:
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

Context:
Consider inlining less complex functions such as the instances below:

```solidity
4 results - 4 files

/LotterySetup.sol
160: function _baseJackpot(uint256 _initialPot) internal view returns (uint256)

/ReferralSystem.sol
156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards)
161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards)

/PercentageMath.sol
 22: function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result)
```
- `_baseJackpot()` only used in `fixedReward()` within the same contract
<br/>
- `playerRewardsPerDraw()` only used in `referralDrawFinalize()` within the same contract
<br/>
- `referrerRewardsPerDraw()` only used in `referralDrawFinalize()` within the same contract
<br/>
- `getPercentageInt()` only used in `calculateExcessPot()` in `LotteryMath.sol`


### [G-09] Inline a `modifer` that is only used once

Description: 
Inline the modifier to save some gas without losing readability in function

Context:
Consider inlining modifiers only used once such as the instances below

```solidity
6 results - 2 files

/LotterySetup.sol
104: modifier requireJackpotInitialized()
112: modifier beforeTicketRegistrationDeadline(uint128 drawId)

/Lottery.sol
 44: modifier requireValidTicket(uint256 ticket)
 52: modifier whenNotExecutingDraw()
 60: modifier onlyWhenExecutingDraw()
 69: modifier onlyTicketOwner(uint256 ticketId)
```
- `requireJackpotInitialized()` only used in `buyTickets()` in `Lottery.sol`
<br/>
- `beforeTicketRegistrationDeadline()` only used in `registerTicket()` in `Lottery.sol`
<br/>
- `requireValidTicket()` only used in `registerTicket()` within the same contract
<br/>
- `whenNotExecutingDraw()` only used in `executeDraw()` within the same contract
<br/>
- `onlyWhenExecutingDraw()` only used in `receiveRandomNumber()` within the same contract
<br/>
- `onlyTicketOwner` only used in `claimWinningTicket()` within the same contract

### [G-10] Check amount before execution of functions for possible gas savings
Context:
```solidity
2 results - 1 file

/StakedTokenLock.sol
24:     function deposit(uint256 amount) external override onlyOwner
34:        stakedToken.transferFrom(msg.sender, address(this), amount);

37:    function withdraw(uint256 amount) external override onlyOwner
47:        stakedToken.transfer(msg.sender, amount)
```


Description:
Before execution of `transfer` functions, we should check if amount is zero to prevent wastage of gas when `transfer` functions do not do anything upon execution


### [G-11] Usage of `uint` smaller than 32 bytes (256 bits) incurs overhead
```solidity
1 result - 1 file

/LotterySetup.sol
170: uint16 reward
```

Description:
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. 


Recommendation:
Replace `uint` smaller than 32 bytes (256 bits) with `uint256` for instances that does not compromise on additional storage slots.