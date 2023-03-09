
## Summary

---

### Low Risk Findings
|Id|Title|Instances|
|:--:|:-------|:--:|
|[L-01]| It's possible to trigger the referral system without buying a ticket | 1 |
|[L-02]| Use of `external` function with `this` instead of defining the function `public` | 1 |
|[L-03]| Use of `mint` instead of `safeMint` in `ERC-721` | 1 |
|[L-04]| Missing events in sensitive functions | 1 |
|[L-05]| Owner can renounce ownership | 4 |

Total: 8 instances over 5 issues.

### Non Critical Findings
|Id|Title|Instances|
|:--:|:-------|:--:|
|[NC-01]| Unused named `return` is confusing | 17 |
|[NC-02]| Use `require` instead of `assert` | 2 |
|[NC-03]| Some functions don't follow the Solidity naming conventions | 40 |
|[NC-04]| Use of floating pragma | 3 |
|[NC-05]| Missing or incomplete NatSpec | 18 |

Total: 80 instances over 5 issues.

## Low Risk Findings

---

### [L-01] It's possible to trigger the referral system without buying a ticket


There is no check to be sure that `tickets` and `drawIds` are both of size zero, so it's possible to trigger the referral system without buying a ticket: this could lead to unexpected scenarios.


*There is 1 instance of this issue.*


```solidity
File: src/Lottery.sol

110: 		    function buyTickets(
111: 		        uint128[] calldata drawIds,
112: 		        uint120[] calldata tickets,
113: 		        address frontend,
114: 		        address referrer
115: 		    )
116: 		        external
117: 		        override
118: 		        requireJackpotInitialized
119: 		        returns (uint256[] memory ticketIds)
120: 		    {
121: 		        if (drawIds.length != tickets.length) {
122: 		            revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
123: 		        }
124: 		        ticketIds = new uint256[](tickets.length);
125: 		        for (uint256 i = 0; i < drawIds.length; ++i) {
126: 		            ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
127: 		        }
128: 		        referralRegisterTickets(currentDraw, referrer, msg.sender, tickets.length);
129: 		        frontendDueTicketSales[frontend] += tickets.length;
130: 		        rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);
131: 		    }

```

### [L-02] Use of `external` function with `this` instead of defining the function `public`


If a function is meant to be called both inside and outside the contract, don't use `external` with `this`, but declare the function `public` instead.


*There is 1 instance of this issue.*


```solidity
File: src/Lottery.sol

261: 		        (claimedAmount, winTier) = this.claimable(ticketId);

```

### [L-03] Use of `mint` instead of `safeMint` in `ERC-721`


An ERC-721 could be bought by a contract that doesn't support it, consider using `safeMint` instead of `mint`.


*There is 1 instance of this issue.*


```solidity
File: src/Ticket.sol

26: 		        _mint(to, ticketId);

```

### [L-04] Missing events in sensitive functions

Events should be emitted when sensitive changes are made to the contracts, but some functions lack them.

*There is 1 instance of this issue.*


```solidity
File: src/staking/Staking.sol

118: 		    function _updateReward(address account) internal {
119: 		        uint256 currentRewardPerToken = rewardPerToken();
120: 		        rewardPerTokenStored = currentRewardPerToken;
121: 		        lastUpdateTicketId = lottery.nextTicketId();
122: 		        rewards[account] = earned(account);
123: 		        userRewardPerTokenPaid[account] = currentRewardPerToken;
124: 		    }

```

### [L-05] Owner can renounce ownership

The contract's owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

Openzeppelin's `Ownable` used in this project implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design.

Renouncing ownership will leave the contract without an Owner, therefore removing any functionality that needs authority.

*There are 4 instances of this issue.*


```solidity
File: src/RNSourceController.sol

77: 		    function initSource(IRNSource rnSource) external override onlyOwner {

89: 		    function swapSource(IRNSource newSource) external override onlyOwner {


File: src/staking/StakedTokenLock.sol

24: 		    function deposit(uint256 amount) external override onlyOwner {

37: 		    function withdraw(uint256 amount) external override onlyOwner {

```

## Non Critical Findings

---

### [NC-01] Unused named `return` is confusing

Declaring named returns, but not using them, is confusing. Consider either completely removing them (by declaring just the type without a name), or remove the return statement and do a variable assignment.

This would improve the readability of the code, and it may also help reduce regressions during future code refactors.

*There are 17 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity
File: src/Lottery.sol

234: 		    function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
235: 		        return drawRewardSize(currentDraw, winTier);

238: 		    function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {
239: 		        return LotteryMath.calculateReward(
240: 		            currentNetProfit,
241: 		            fixedReward(winTier),
242: 		            fixedReward(selectionSize),
243: 		            ticketsSold[drawId],
244: 		            winTier == selectionSize,
245: 		            expectedPayout
246: 		        );


File: src/LotterySetup.sol

120: 		    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
122: 		            return _baseJackpot(initialPot);

120: 		    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
124: 		            return 0;

120: 		    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
128: 		            return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));


File: src/PercentageMath.sol

17: 		    function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
18: 		        return number * percentage / PERCENTAGE_BASE;

22: 		    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
23: 		        return number * int256(percentage) / int256(PERCENTAGE_BASE);


File: src/ReferralSystem.sol

115: 		        returns (uint256 minimumEligible)
119: 		            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT);

115: 		        returns (uint256 minimumEligible)
123: 		            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);

115: 		        returns (uint256 minimumEligible)
127: 		            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);

115: 		        returns (uint256 minimumEligible)
129: 		        return 5000;

156: 		    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
158: 		        return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;

161: 		    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
162: 		        return rewardsToReferrersPerDraw[Math.min(rewardsToReferrersPerDraw.length - 1, drawId)];


File: src/TicketUtils.sol

24: 		        returns (bool isValid)
32: 		            return (ticketSize == selectionSize) && (ticket == uint256(0));


File: src/staking/Staking.sol

48: 		    function rewardPerToken() public view override returns (uint256 _rewardPerToken) {
51: 		            return rewardPerTokenStored;

48: 		    function rewardPerToken() public view override returns (uint256 _rewardPerToken) {
58: 		        return rewardPerTokenStored + (unclaimedRewards * 1e18 / _totalSupply);

61: 		    function earned(address account) public view override returns (uint256 _earned) {
62: 		        return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];

```
</details>

---
### [NC-02] Use `require` instead of `assert`

Prior to Solidity 0.8.0, the big difference between the `assert` and `require` is that the former uses up all the remaining gas and reverts all the changes made when the predicate is false.

It should be avoided even after Solidity version 0.8.0 as the documentation states:

> Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix.

*There are 2 instances of this issue.*


```solidity
File: src/LotterySetup.sol

147: 		        assert(initialPot > 0);


File: src/TicketUtils.sol

99: 		            assert((winTier <= selectionSize) && (intersection == uint256(0)));

```

### [NC-03] Some functions don't follow the Solidity naming conventions

Follow the Solidity naming convention by adding an underscore before the function name, for any `private` and `internal` functions.

*There are 40 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity
File: src/Lottery.sol

181: 		    function registerTicket(

203: 		    function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {

238: 		    function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {

249: 		    function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {

259: 		    function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {

271: 		    function returnUnclaimedJackpotToThePot() private {

279: 		    function requireFinishedDraw(uint128 drawId) internal view override {

285: 		    function mintNativeTokens(address mintTo, uint256 amount) internal override {


File: src/LotteryMath.sol

35: 		    function calculateNewProfit(

62: 		    function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot) {

73: 		    function calculateMultiplier(

96: 		    function calculateReward(

119: 		    function calculateRewards(


File: src/LotterySetup.sol

164: 		    function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {


File: src/PercentageMath.sol

17: 		    function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {

22: 		    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {


File: src/ReferralSystem.sol

52: 		    function referralRegisterTickets(

74: 		    function mintNativeTokens(address mintTo, uint256 amount) internal virtual;

87: 		    function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {

111: 		    function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)

134: 		    function requireFinishedDraw(uint128 drawId) internal view virtual;

136: 		    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {

156: 		    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

161: 		    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {


File: src/RNSourceBase.sol

33: 		    function fulfill(uint256 requestId, uint256 randomNumber) internal {

48: 		    function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId);


File: src/RNSourceController.sol

38: 		    function requestRandomNumber() internal {

58: 		    function receiveRandomNumber(uint256 randomNumber) internal virtual;

106: 		    function requestRandomNumberFromSource() private {


File: src/Ticket.sol

19: 		    function markAsClaimed(uint256 ticketId) internal {

23: 		    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {


File: src/TicketUtils.sol

17: 		    function isValidTicket(

43: 		    function reconstructTicket(

83: 		    function ticketWinTier(


File: src/VRFv2RNSource.sol

28: 		    function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {

32: 		    function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {


File: script/config/LotteryConfig.sol

11: 		    function getLottery(


File: script/config/ReferralSystemConfig.sol

12: 		    function getLotteryRewardsData()


File: script/config/RewardTokenConfig.sol

10: 		    function getRewardToken() internal returns (IERC20 token) {


File: script/config/RNSourceConfig.sol

11: 		    function getRNSource(address authorizedConsumer) internal returns (IRNSource rnSource) {

```
</details>

---
### [NC-04] Use of floating pragma

Locking the pragma helps avoid accidental deploys with an outdated compiler version that may introduce bugs and unexpected vulnerabilities.

Floating pragma is meant to be used for libraries and contracts that have external users and need backward compatibility.

*There are 3 instances of this issue.*


```solidity
File: src/VRFv2RNSource.sol

3: 		pragma solidity ^0.8.7;


File: src/interfaces/IVRFv2RNSource.sol

3: 		pragma solidity ^0.8.7;


File: src/staking/StakedTokenLock.sol

3: 		pragma solidity ^0.8.17;

```

### [NC-05] Missing or incomplete NatSpec

Some functions miss a NatSpec, or they have an incomplete one.
It is recommended that contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI).

Include either `@notice` and/or `@dev` to describe how the function is supposed to work, a `@param` to describe each parameter, and a `@return` for any return values.

*There are 18 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity
File: src/Lottery.sol

110: 		    function buyTickets(

133: 		    function executeDraw() external override whenNotExecutingDraw {

151: 		    function claimRewards(LotteryRewardType rewardType) external override returns (uint256 claimedAmount) {

170: 		    function claimWinningTickets(uint256[] calldata ticketIds) external override returns (uint256 claimedAmount) {


File: src/LotterySetup.sol

132: 		    function finalizeInitialPotRaise() external override {


File: src/LotteryToken.sol

22: 		    function mint(address account, uint256 amount) external override {


File: src/ReferralSystem.sol

76: 		    function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {


File: src/RNSourceController.sol

46: 		    function onRandomNumberFulfilled(uint256 randomNumber) external override {

60: 		    function retry() external override {

77: 		    function initSource(IRNSource rnSource) external override onlyOwner {

89: 		    function swapSource(IRNSource newSource) external override onlyOwner {


File: src/staking/StakedTokenLock.sol

24: 		    function deposit(uint256 amount) external override onlyOwner {

37: 		    function withdraw(uint256 amount) external override onlyOwner {

50: 		    function getReward() external override {


File: src/staking/Staking.sol

65: 		    /* ========== MUTATIVE FUNCTIONS ========== */
66: 		
67: 		    function stake(uint256 amount) external override {

79: 		    function withdraw(uint256 amount) public override {

91: 		    function getReward() public override {

103: 		    function exit() external override {

```
</details>

---