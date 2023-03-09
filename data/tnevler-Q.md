# Report
## Low Issues ##
### [L-1]: Incorrect comment
**Context:**
```/// @param rewardType 0 - staking reward, 1 - frontend reward.``` [L72](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol#L72)

**Recommendation:**
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol#L44
It should be like this:
```/// @param rewardType 0 - frontend reward, 1 - staking reward.```

### [L-2]: Add check it is not less than the minimum
**Context:**
1. ```if (_maxFailedAttempts > MAX_MAX_FAILED_ATTEMPTS)``` [L27](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L27)
1. ```if (_maxRequestDelay > MAX_REQUEST_DELAY``` [L30](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L30)

**Recommendation:**
Check that _maxFailedAttempts and _maxRequestDelay is not too small.

### [L-3]:  Incorrect constant name
**Context:**
```uint256 private constant MAX_MAX_FAILED_ATTEMPTS = 10;``` [L19](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L19)

**Recommendation:**
Change to ```uint256 private constant MAX_FAILED_ATTEMPTS = 10;```

### [L-4]: Invalid parameter type
**Context:**
1. ```modifier requireValidTicket(uint256 ticket) {``` [L44](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L44) (Change to uint120. [Here](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L189) ticket is uint120)
1. ```uint256 ticket,``` [L18](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L18) (Change to uint120)

**Description:**
According [TicketUtils.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L6) ```/// Ticket is represented as uint120 packed ticket:```

### [L-5]: Check if nonJackpotFixedRewards is correct
**Context:**
```nonJackpotFixedRewards = packFixedRewards(lotterySetupParams.fixedRewards);``` [L91](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L91)
**Description:**
The user can make a mistake in the order of rewards. 
**Recommendation:**
Check that each next reward is greater than the previous one.

### [L-6]:  Unsafe casting may overflow
**Context:**
1. ```uint16 reward = uint16(rewards[winTier] / divisor);``` [L170](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L170)
1. ```currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);``` [L275](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275)
1. ```unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);``` [L69](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L69)
1. ```unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);``` [L71](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L71)
1. ```newProfit = oldProfit + int256(ticketsSalesToPot);``` [L48](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L48)
1. ```newProfit -= int256(expectedRewardsOut);``` [L55](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L55)
1. ```excessPotInt -= int256(fixedJackpotSize);``` [L64](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L64)
1. ```excessPot = excessPotInt > 0 ? uint256(excessPotInt) : 0;``` [L65](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L65)

**Description:**
While Solidity 0.8.x checks for overflows on arithmetic operations, it does not do so for casting.

**Recommendation:**
Use OpenZeppelin’s SafeCast library to prevent unexpected overflows.

## Non-Critical Issues ##
### [N-1]: Function defines a named return variable but then it uses return statements
**Context:**

1. ```return rewardPerTokenStored;``` [L51](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L51) 
1. ```return rewardPerTokenStored + (unclaimedRewards * 1e18 / _totalSupply);``` [L58](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L58) 
1. ```return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];``` [L62](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L62) 
1. ```return _baseJackpot(initialPot);``` [L122](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L122) 
1. ```return 0;``` [L124](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L124) 
1. ```return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));``` [L128](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L128) 
1. ```return drawRewardSize(currentDraw, winTier);``` [L235](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L235) 
1. ```return LotteryMath.calculateReward(``` [L239](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L239) 
1. ```return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT);``` [L119](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L119) 
1. ```return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);``` [L123](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L123) 
1. ```return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);``` [L127](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L127) 
1. ```return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;``` [L158](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L158) 
1. ```return rewardsToReferrersPerDraw[Math.min(rewardsToReferrersPerDraw.length - 1, drawId)];``` [L162](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L162) 
1. ```return number * percentage / PERCENTAGE_BASE;``` [L18](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L18) 
1. ```return number * int256(percentage) / int256(PERCENTAGE_BASE);``` [L23](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L23) 
1. ```return (ticketSize == selectionSize) && (ticket == uint256(0));``` [L32](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L32) 

**Recommendation:**

Choose named return variable or return statement. It is unnecessary to use both.

### [N-2]: Follow Solidity standard naming conventions
**Context:**

1. ```function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {``` [L28](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L28) (requestRandomnessFromUnderlyingSource should start with_)
1. ```function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {``` [L32](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L32) (fulfillRandomWords should start with_)
1. ```uint256 internal immutable firstDrawSchedule;``` [L26](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L26) (firstDrawSchedule should start with_)
1. ```uint256 private immutable nonJackpotFixedRewards;``` [L34](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L34) (nonJackpotFixedRewards should start with_)
1. ```uint256 private constant BASE_JACKPOT_PERCENTAGE = 30_030; // 30.03%``` [L36](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L36) (BASE_JACKPOT_PERCENTAGE should start with_)
1. ```function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {``` [L164](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L164) (packFixedRewards should start with_)
1. ```uint256 private claimedStakingRewardAtTicketId;``` [L25](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L25) (claimedStakingRewardAtTicketId should start with_)
1. ```mapping(address => uint256) private frontendDueTicketSales;``` [L26](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L26) (frontendDueTicketSales should start with_)
1. ```mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;``` [L27](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L27) (unclaimedCount should start with_)
1. ```function registerTicket(``` [L181](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L181) (registerTicket should start with_)
1. ```function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {``` [L203](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L203) (receiveRandomNumber should start with_)
1. ```function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {``` [L238](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L238) (drawRewardSize should start with_)
1. ```function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {``` [L249](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L249) (dueTicketsSoldAndReset should start with_)
1. ```function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {``` [L259](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L259) (claimWinningTicket should start with_)
1. ```function returnUnclaimedJackpotToThePot() private {``` [L271](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L271) (returnUnclaimedJackpotToThePot should start with_)
1. ```function requireFinishedDraw(uint128 drawId) internal view override {``` [L279](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L279) (requireFinishedDraw should start with_)
1. ```function mintNativeTokens(address mintTo, uint256 amount) internal override {``` [L285](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L285) (mintNativeTokens should start with_)
1. ```function markAsClaimed(uint256 ticketId) internal {``` [L19](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L19) (markAsClaimed should start with_)
1. ```address internal immutable authorizedConsumer;``` [L8](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L8) (authorizedConsumer should start with _)
1. ```mapping(uint256 => RandomnessRequest) internal requests;``` [L9](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L9) (requests should start with _)
1. ```function fulfill(uint256 requestId, uint256 randomNumber) internal {``` [L33](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L33) (fulfill should start with _)
1. ```function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId);``` [L48](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L48) (requestRandomnessFromUnderlyingSource should start with _)
1. ```uint256 private constant MAX_MAX_FAILED_ATTEMPTS = 10;``` [L19](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L19) (MAX_MAX_FAILED_ATTEMPTS should start with_)
1. ```uint256 private constant MAX_REQUEST_DELAY = 5 hours;``` [L20](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L20) (MAX_REQUEST_DELAY should start with_)
1. ```function requestRandomNumber() internal {``` [L38](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L38) (requestRandomNumber should start with_)
1. ```function receiveRandomNumber(uint256 randomNumber) internal virtual;``` [L58](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L58) (receiveRandomNumber should start with_)
1. ```function requestRandomNumberFromSource() private {``` [L106](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L106) (requestRandomNumberFromSource should start with_)
1. ```function referralRegisterTickets(``` [L52](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L52) (referralRegisterTickets should start with_)
1. ```function mintNativeTokens(address mintTo, uint256 amount) internal virtual;``` [L74](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L74) (mintNativeTokens should start with_)
1. ```function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {``` [L87](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L87) (referralDrawFinalize should start with_)
1. ```function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)``` [L111](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L111) (getMinimumEligibleReferralsFactorCalculation should start with_)
1. ```function requireFinishedDraw(uint128 drawId) internal view virtual;``` [L134](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L134) (requireFinishedDraw should start with_)
1. ```function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {``` [L136](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L136) (claimPerDraw should start with_)
1. ```function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {``` [L156](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L156) (playerRewardsPerDraw should start with_)
1. ```function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {``` [L161](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L161) (referrerRewardsPerDraw should start with_)

**Description:**

The above codes don't follow [Solidity's standard naming convention](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions).  
- Internal and private functions should use mixedCase format starting with an underscore.
- Structs should be named using the CapWords style. Examples: MyCoin, Position, PositionXY.
- Events should be named using the CapWords style. Examples: Deposit, Transfer, Approval, BeforeTransfer, AfterTransfer.
- Functions should use mixedCase. Examples: getBalance, transfer, verifyOwner, addMember, changeOwner.
- Function arguments should use mixedCase. Examples: initialSupply, account, recipientAddress, senderAddress, newOwner.
- Local and State Variable should use mixedCase. Examples: totalSupply, remainingSupply, balancesOf, creatorAddress, isPreSale, tokenExchangeRate.
- Constants should be named with all capital letters with underscores separating words. Examples: MAX_BLOCKS, TOKEN_NAME, TOKEN_TICKER, CONTRACT_VERSION.
- Modifier should use mixedCase. Examples: onlyBy, onlyAfter, onlyDuringThePreSale.
- Enums, in the style of simple type declarations, should be named using the CapWords style. Examples: TokenGroup, Frame, HashStyle, CharacterLocation.


### [N-3]: Wrong order of functions
**Context:**

1. ```modifier requireJackpotInitialized() {``` [L104](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L104) (modifier definition can not go after constructor)
1. ```function onRandomNumberFulfilled(uint256 randomNumber) external override {``` [L46](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L46) (external function can not go after intenal function)
1. ```function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {``` [L76](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L76) (external function can not go after internal function)

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) functions should be grouped according to their visibility and ordered:

+ constructor

+ receive function (if exists)

+ fallback function (if exists)

+ external

+ public

+ internal

+ private

Within a grouping, place the view and pure functions last.

**Recommendation:**

Put the functions in the correct order according to the documentation.

### [N-4]: add check >0 to avoid zero transfer
**Context:**

1. ```stakedToken.transferFrom(msg.sender, address(this), amount);``` [L34](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L34) (check amount > 0)
1. ```stakedToken.transfer(msg.sender, amount);``` [L47](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L47) (check amount > 0)
1. ```rewardToken.safeTransfer(beneficiary, claimedAmount);``` [L156](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L156) (check claimedAmount > 0)
1. ```rewardToken.safeTransfer(msg.sender, claimedAmount);``` [L175](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L175) (check claimedAmount > 0)
