## Non Critical Issues

|       | Issue                                                                          |
| ----- | :----------------------------------------------------------------------------- |
| NC-1  | ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS                                  |
| NC-2  | ADD TO BLACKLIST FUNCTION                                                      |
| NC-3  | GENERATE PERFECT CODE HEADERS EVERY TIME                                       |
| NC-4  | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS                                |
| NC-5  | SOLIDITY COMPILER VERSIONS MISMATCH                                            |
| NC-6  | FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES                        |
| NC-7  | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION                  |
| NC-8  | NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS                              |
| NC-9  | NO SAME VALUE INPUT CONTROL                                                    |
| NC-10 | ADD PARAMETER TO EVENT-EMIT                                                    |
| NC-11 | FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE            |
| NC-12 | DON’T USE PERIODS WITH FRAGMENTS                                               |
| NC-13 | USE A MORE RECENT VERSION OF SOLIDITY                                          |
| NC-14 | FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS        |
| NC-15 | LINES ARE TOO LONG or LONG LINES ARE NOT SUITABLE FOR THE SOLIDITY STYLE GUIDE |
| NC-16 | USE UNDERSCORES FOR NUMBER LITERALS                                            |

### [NC-1] ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS

#### Description:

Add to indexed parameter for countable Events

[code4arena example](https://code4rena.com/reports/2022-11-stakehouse#n-10-add-to-indexed-parameter-for-countable-events)

```solidity
File: src/interfaces/ILottery.sol

73:     event ClaimedRewards(address indexed rewardRecipient, uint256 indexed amount, LotteryRewardType indexed rewardType);

```

#### Recommendation:

Add Event-Emit

### [NC-2] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Manifold Contract
https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-4] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS

#### Description:

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes.

Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

#### **Proof Of Concept**

```solidity
File: src/LotteryMath.sol

14:     uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;

16:     uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;

18:     uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;

20:     uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;

22:     uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;

24:     uint128 public constant DRAWS_PER_YEAR = 52;

```

```solidity
File: src/LotteryToken.sol

12:     uint256 public constant override INITIAL_SUPPLY = 1_000_000_000e18;

```

```solidity
File: src/PercentageMath.sol

8:     uint256 public constant PERCENTAGE_BASE = 100_000;

11:     uint256 public constant ONE_PERCENT = 1000;

```

### [NC-5] SOLIDITY COMPILER VERSIONS MISMATCH

#### Description:

The project is compiled with different versions of Solidity, which is not recommended because it can lead to undefined behaviors.

It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/LotteryMath.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/LotterySetup.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/LotteryToken.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/PercentageMath.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/RNSourceBase.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/RNSourceController.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/ReferralSystem.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/Ticket.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/TicketUtils.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/VRFv2RNSource.sol

3: pragma solidity ^0.8.7;

```

```solidity
File: src/interfaces/ILottery.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILotterySetup.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILotteryToken.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/IRNSource.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/IRNSourceController.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/IReferralSystem.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/IReferralSystemDynamic.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ITicket.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/IVRFv2RNSource.sol

3: pragma solidity ^0.8.7;

```

```solidity
File: src/staking/StakedTokenLock.sol

3: pragma solidity ^0.8.17;

```

```solidity
File: src/staking/Staking.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/staking/interfaces/IStakedTokenLock.sol

3: pragma solidity 0.8.19;

```

```solidity
File: src/staking/interfaces/IStaking.sol

3: pragma solidity 0.8.19;

```

### [NC-6] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

#### Description:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

5: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

6: import "@openzeppelin/contracts/utils/math/Math.sol";

7: import "src/ReferralSystem.sol";

8: import "src/RNSourceController.sol";

9: import "src/staking/Staking.sol";

10: import "src/LotterySetup.sol";

11: import "src/TicketUtils.sol";

```

```solidity
File: src/LotteryMath.sol

5: import "src/interfaces/ILottery.sol";

6: import "src/PercentageMath.sol";

```

```solidity
File: src/LotterySetup.sol

5: import "@openzeppelin/contracts/utils/math/Math.sol";

6: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

7: import "src/PercentageMath.sol";

8: import "src/LotteryToken.sol";

9: import "src/interfaces/ILotterySetup.sol";

10: import "src/Ticket.sol";

```

```solidity
File: src/LotteryToken.sol

5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

6: import "src/interfaces/ILotteryToken.sol";

7: import "src/LotteryMath.sol";

```

```solidity
File: src/RNSourceBase.sol

5: import "src/interfaces/IRNSource.sol";

```

```solidity
File: src/RNSourceController.sol

5: import "@openzeppelin/contracts/access/Ownable2Step.sol";

6: import "src/interfaces/IRNSource.sol";

7: import "src/interfaces/IRNSourceController.sol";

```

```solidity
File: src/ReferralSystem.sol

5: import "@openzeppelin/contracts/utils/math/Math.sol";

6: import "src/interfaces/IReferralSystem.sol";

7: import "src/PercentageMath.sol";

```

```solidity
File: src/Ticket.sol

5: import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

6: import "src/interfaces/ITicket.sol";

```

```solidity
File: src/VRFv2RNSource.sol

5: import "@chainlink/contracts/src/v0.8/VRFV2WrapperConsumerBase.sol";

6: import "src/interfaces/IVRFv2RNSource.sol";

7: import "src/RNSourceBase.sol";

```

```solidity
File: src/interfaces/ILottery.sol

5: import "src/interfaces/ILotterySetup.sol";

6: import "src/interfaces/IRNSourceController.sol";

7: import "src/interfaces/ITicket.sol";

8: import "src/interfaces/IReferralSystem.sol";

```

```solidity
File: src/interfaces/ILotterySetup.sol

5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

6: import "src/interfaces/ITicket.sol";

```

```solidity
File: src/interfaces/ILotteryToken.sol

5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```

```solidity
File: src/interfaces/IRNSourceController.sol

5: import "@openzeppelin/contracts/access/Ownable2Step.sol";

6: import "src/interfaces/IRNSource.sol";

```

```solidity
File: src/interfaces/IReferralSystem.sol

5: import "src/interfaces/ILotteryToken.sol";

```

```solidity
File: src/interfaces/ITicket.sol

5: import "@openzeppelin/contracts/token/ERC721/IERC721.sol";

```

```solidity
File: src/interfaces/IVRFv2RNSource.sol

5: import "src/interfaces/IRNSource.sol";

```

```solidity
File: src/staking/StakedTokenLock.sol

5: import "@openzeppelin/contracts/access/Ownable2Step.sol";

6: import "src/staking/interfaces/IStakedTokenLock.sol";

7: import "src/staking/interfaces/IStaking.sol";

```

```solidity
File: src/staking/Staking.sol

5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

6: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

7: import "src/interfaces/ILottery.sol";

8: import "src/LotteryMath.sol";

9: import "src/staking/interfaces/IStaking.sol";

```

```solidity
File: src/staking/interfaces/IStakedTokenLock.sol

5: import "src/staking/interfaces/IStaking.sol";

```

```solidity
File: src/staking/interfaces/IStaking.sol

5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

6: import "src/interfaces/ILottery.sol";

```

#### Recommended Mitigation Steps:

`import {contract1 , contract2} from "filename.sol";` OR Use specific imports syntax per solidity docs recommendation.

### [NC-7] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### Description:

https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/

[code4arena example](https://code4rena.com/reports/2022-10-thegraph#n-15--for-extended-using-for-usage-use-the-latest-pragma-version)

#### **Proof Of Concept**

```solidity
File: src/VRFv2RNSource.sol

3: pragma solidity ^0.8.7;

```

```solidity
File: src/interfaces/IVRFv2RNSource.sol

3: pragma solidity ^0.8.7;

```

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

### [NC-8] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

### Context:

All Contracts

#### Description:

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html.

#### Recommended Mitigation Steps:

NatSpec comments should be increased in contracts

### [NC-9] NO SAME VALUE INPUT CONTROL

#### Recommended Mitigation Steps:

Add code like this; `if (oracle == _oracle revert ADDRESS_SAME();`

#### **Proof Of Concept**

```solidity
File: src/RNSourceBase.sol

12:         authorizedConsumer = _authorizedConsumer;

```

```solidity
File: src/RNSourceController.sol

33:         maxFailedAttempts = _maxFailedAttempts;

34:         maxRequestDelay = _maxRequestDelay;

```

```solidity
File: src/ReferralSystem.sol

41:         rewardsToReferrersPerDraw = _rewardsToReferrersPerDraw;

43:         playerRewardFirstDraw = _playerRewardFirstDraw;

44:         playerRewardDecreasePerDraw = _playerRewardDecreasePerDraw;

```

```solidity
File: src/VRFv2RNSource.sol

23:         requestConfirmations = _requestConfirmations;

24:         callbackGasLimit = _callbackGasLimit;

```

```solidity
File: src/staking/StakedTokenLock.sol

20:         depositDeadline = _depositDeadline;

21:         lockDuration = _lockDuration;

```

```solidity
File: src/staking/Staking.sol

41:         lottery = _lottery;

42:         rewardsToken = _rewardsToken;

43:         stakingToken = _stakingToken;

```

### [NC-10] ADD PARAMETER TO EVENT-EMIT

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible.

Events with no old value:

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

141:         emit StartedExecutingDraw(currentDraw);

```

```solidity
File: src/LotterySetup.sol

149:         emit InitialPotPeriodFinalized(raised);

```

```solidity
File: src/RNSourceController.sol

86:         emit SourceSet(rnSource);

102:         emit SourceSet(newSource);

113:             emit SuccessfulRNRequest(source);

```

### [NC-11] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private, within a grouping, place the view and pure functions last.

### [NC-12] DON’T USE PERIODS WITH FRAGMENTS

#### Context:

All Contracts

#### Description:

Throughout the files, some of the comments have fragments that end with periods and some of them end without periods. It should be consistent. They should either be converted to actual sentences with both a noun phrase and a verb phrase, or the periods should be removed.

#### **Proof Of Concept**

One example:

```solidity
File: src/Lottery.sol

42:         /// @dev Checks if ticket is a valid ticket, and reverts if invalid
51:         /// @dev Checks if we are not executing draw already.

```

### [NC-13] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: src/VRFv2RNSource.sol

3: pragma solidity ^0.8.7;

```

```solidity
File: src/interfaces/IVRFv2RNSource.sol

3: pragma solidity ^0.8.7;

```

```solidity
File: src/staking/StakedTokenLock.sol

3: pragma solidity ^0.8.17;

```

### [NC-14] FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

#### Description:

Solidity’s standard naming convention for internal and private functions and variables (apart from constants): the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

Solidity’s standard naming convention for internal and private constants variables: the snake_case format starting with an underscore (\_mixedCase starting with an underscore) and use ALL_CAPS for naming them.

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

25:     uint256 private claimedStakingRewardAtTicketId;

26:     mapping(address => uint256) private frontendDueTicketSales;

27:     mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;

181:     function registerTicket(

203:     function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {

238:     function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {

249:     function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {

259:     function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {

271:     function returnUnclaimedJackpotToThePot() private {

279:     function requireFinishedDraw(uint128 drawId) internal view override {

285:     function mintNativeTokens(address mintTo, uint256 amount) internal override {

```

```solidity
File: src/LotteryMath.sol

35:     function calculateNewProfit(

62:     function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot) {

73:     function calculateMultiplier(

96:    function calculateReward(

119:   function calculateRewards(

```

```solidity
File: src/LotterySetup.sol

26:     uint256 internal immutable firstDrawSchedule;

34:     uint256 private immutable nonJackpotFixedRewards;

164:     function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {

```

```solidity
File: src/PercentageMath.sol

17:     function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {

22:     function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {

```

```solidity
File: src/RNSourceBase.sol

8:     address internal immutable authorizedConsumer;

9:     mapping(uint256 => RandomnessRequest) internal requests;

33:     function fulfill(uint256 requestId, uint256 randomNumber) internal {

48:     function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId);

```

```solidity
File: src/RNSourceController.sol

38:     function requestRandomNumber() internal {

58:     function receiveRandomNumber(uint256 randomNumber) internal virtual;

106:     function requestRandomNumberFromSource() private {

```

```solidity
File: src/ReferralSystem.sol

52:     function referralRegisterTickets(

74:     function mintNativeTokens(address mintTo, uint256 amount) internal virtual;

87:     function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {

111:     function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)

134:     function requireFinishedDraw(uint128 drawId) internal view virtual;

136:     function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {

156:     function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

161:     function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

```

```solidity
File: src/Ticket.sol

19:     function markAsClaimed(uint256 ticketId) internal {

23:     function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {

```

```solidity
File: src/TicketUtils.sol

17:     function isValidTicket(

43:     function reconstructTicket(

83:     function ticketWinTier(

```

```solidity
File: src/VRFv2RNSource.sol

28:     function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {

32:     function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {

```

### [NC-15] LINES ARE TOO LONG or LONG LINES ARE NOT SUITABLE FOR THE SOLIDITY STYLE GUIDE

#### Description:

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

[Reference](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length)

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

21: contract Lottery is ILottery, Ticket, LotterySetup, ReferralSystem, RNSourceController {

94:         ReferralSystem(playerRewardFirstDraw, playerRewardDecreasePerDraw, rewardsToReferrersPerDraw)

107:         nativeToken.safeTransfer(msg.sender, ILotteryToken(address(nativeToken)).INITIAL_SUPPLY());

126:             ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);

128:         referralRegisterTickets(currentDraw, referrer, msg.sender, tickets.length);

130:         rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);

144:     function unclaimedRewards(LotteryRewardType rewardType) external view override returns (uint256 rewards) {

148:         rewards = LotteryMath.calculateRewards(ticketPrice, dueTicketsSold, rewardType);

151:     function claimRewards(LotteryRewardType rewardType) external override returns (uint256 claimedAmount) {

152:         address beneficiary = (rewardType == LotteryRewardType.FRONTEND) ? msg.sender : stakingRewardRecipient;

153:         claimedAmount = LotteryMath.calculateRewards(ticketPrice, dueTicketsSoldAndReset(beneficiary), rewardType);

159:     function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier) {

163:             winTier = TicketUtils.ticketWinTier(ticketInfo.combination, _winningTicket, selectionSize, selectionMax);

164:             if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {

170:     function claimWinningTickets(uint256[] calldata ticketIds) external override returns (uint256 claimedAmount) {

195:         emit NewTicket(currentDraw, ticketId, drawId, msg.sender, ticket, frontend, referrer);

203:     function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {

204:         uint120 _winningTicket = TicketUtils.reconstructTicket(randomNumber, selectionSize, selectionMax);

209:             winAmount[drawFinalized][selectionSize] = drawRewardSize(drawFinalized, selectionSize) / jackpotWinners;

212:                 winAmount[drawFinalized][winTier] = drawRewardSize(drawFinalized, winTier);

231:         emit FinishedExecutingDraw(drawFinalized, randomNumber, _winningTicket);

234:     function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {

238:     function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {

249:     function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {

259:     function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {

266:         unclaimedCount[ticketsInfo[ticketId].drawId][ticketsInfo[ticketId].combination]--;

274:             uint256 unclaimedJackpotTickets = unclaimedCount[drawId][winningTicket[drawId]];

275:             currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);

285:     function mintNativeTokens(address mintTo, uint256 amount) internal override {

```

```solidity
File: src/LotteryMath.sol

18:     uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;

22:     uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;

47:         uint256 ticketsSalesToPot = (ticketsSold * ticketPrice).getPercentage(TICKET_PRICE_TO_POT);

51:             ? calculateReward(oldProfit, fixedJackpotSize, fixedJackpotSize, ticketsSold, true, expectedPayout)

52:             : calculateMultiplier(calculateExcessPot(oldProfit, fixedJackpotSize), ticketsSold, expectedPayout)

62:     function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot) {

84:             bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);

111:             : fixedReward.getPercentage(calculateMultiplier(excess, ticketsSold, expectedPayout));

128:         uint256 rewardPercentage = (rewardType == LotteryRewardType.FRONTEND) ? FRONTEND_REWARD : STAKING_REWARD;

129:         dueRewards = (ticketsSold * ticketPrice).getPercentage(rewardPercentage);

```

```solidity
File: src/LotterySetup.sol

55:             lotterySetupParams.expectedPayout < lotterySetupParams.ticketPrice / 100

56:                 || lotterySetupParams.expectedPayout >= lotterySetupParams.ticketPrice

61:             lotterySetupParams.selectionSize > 16 || lotterySetupParams.selectionSize >= lotterySetupParams.selectionMax

66:             lotterySetupParams.drawSchedule.drawCoolDownPeriod >= lotterySetupParams.drawSchedule.drawPeriod

67:                 || lotterySetupParams.drawSchedule.firstDrawScheduledAt < lotterySetupParams.drawSchedule.drawPeriod

72:             lotterySetupParams.drawSchedule.firstDrawScheduledAt - lotterySetupParams.drawSchedule.drawPeriod;

74:         if (initialPotDeadline < (block.timestamp + lotterySetupParams.drawSchedule.drawPeriod)) {

79:         uint256 tokenUnit = 10 ** IERC20Metadata(address(lotterySetupParams.token)).decimals();

83:         firstDrawSchedule = lotterySetupParams.drawSchedule.firstDrawScheduledAt;

85:         drawCoolDownPeriod = lotterySetupParams.drawSchedule.drawCoolDownPeriod;

91:         nonJackpotFixedRewards = packFixedRewards(lotterySetupParams.fixedRewards);

120:     function fixedReward(uint8 winTier) public view override returns (uint256 amount) {

127:             uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);

128:             return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));

152:     function drawScheduledAt(uint128 drawId) public view override returns (uint256 time) {

156:     function ticketRegistrationDeadline(uint128 drawId) public view override returns (uint256 time) {

160:     function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {

161:         return Math.min(_initialPot.getPercentage(BASE_JACKPOT_PERCENTAGE), jackpotBound);

164:     function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {

168:         uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1);

```

```solidity
File: src/PercentageMath.sol

17:     function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {

22:     function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {

```

```solidity
File: src/RNSourceBase.sol

27:         requests[requestId] = RandomnessRequest({ status: RequestStatus.Pending, randomNumber: 0 });

43:         IRNSourceConsumer(authorizedConsumer).onRandomNumberFulfilled(randomNumber);

48:     function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId);

```

```solidity
File: src/RNSourceController.sol

93:         bool notEnoughRetryInvocations = failedSequentialAttempts < maxFailedAttempts;

94:         bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;

95:         if (notEnoughRetryInvocations || notEnoughTimeReachingMaxFailedAttempts) {

```

```solidity
File: src/ReferralSystem.sol

17:     mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;

19:     mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;

21:     mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;

23:     mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;

62:             if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {

63:                 if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {

65:                         unclaimedTickets[currentDraw][referrer].referrerTicketCount;

67:                 totalTicketsForReferrersPerDraw[currentDraw] += numberOfTickets;

69:             unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

71:         unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);

76:     function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {

87:     function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {

94:             getMinimumEligibleReferralsFactorCalculation(ticketsSoldDuringDraw);

97:         uint256 totalTicketsForReferrersPerCurrentDraw = totalTicketsForReferrersPerDraw[drawFinalized];

105:             playerRewardsPerDrawForOneTicket[drawFinalized] = playerRewardForDraw / ticketsSoldDuringDraw;

108:         emit CalculatedRewardsForDraw(drawFinalized, referrerRewardForDraw, playerRewardForDraw);

111:     function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)

119:             return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT);

123:             return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);

127:             return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);

136:     function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {

139:         UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];

140:         if (_unclaimedTickets.referrerTicketCount >= minimumEligibleReferrals[drawId]) {

141:             claimedReward = referrerRewardPerDrawForOneTicket[drawId] * _unclaimedTickets.referrerTicketCount;

147:             claimedReward += playerRewardsPerDrawForOneTicket[drawId] * _unclaimedTickets.playerTicketCount;

156:     function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

158:         return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;

161:     function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

162:         return rewardsToReferrersPerDraw[Math.min(rewardsToReferrersPerDraw.length - 1, drawId)];

```

```solidity
File: src/Ticket.sol

23:     function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {

```

```solidity
File: src/VRFv2RNSource.sol

9: contract VRFv2RNSource is IVRFv2RNSource, RNSourceBase, VRFV2WrapperConsumerBase {

28:     function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {

29:         requestId = requestRandomness(callbackGasLimit, requestConfirmations, 1);

32:     function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {

34:             revert WrongRandomNumberCountReceived(requestId, randomWords.length);

```

```solidity
File: src/interfaces/ILottery.sol

50: interface ILottery is ITicket, ILotterySetup, IRNSourceController, IReferralSystem {

73:     event ClaimedRewards(address indexed rewardRecipient, uint256 indexed amount, LotteryRewardType indexed rewardType);

79:     event ClaimedTicket(address indexed user, uint256 indexed ticketId, uint256 indexed amount);

89:     event FinishedExecutingDraw(uint128 indexed drawId, uint256 indexed randomNumber, uint120 indexed winningTicket);

92:     function stakingRewardRecipient() external view returns (address rewardRecipient);

104:     function winAmount(uint128 drawId, uint8 winTier) external view returns (uint256 amount);

109:     function currentRewardSize(uint8 winTier) external view returns (uint256 rewardSize);

120:     function unclaimedRewards(LotteryRewardType rewardType) external view returns (uint256 rewards);

128:     function winningTicket(uint128 drawId) external view returns (uint120 winningCombination);

152:     function claimRewards(LotteryRewardType rewardType) external returns (uint256 claimedAmount);

159:     function claimWinningTickets(uint256[] calldata ticketIds) external returns (uint256 claimedAmount);

165:     function claimable(uint256 ticketId) external view returns (uint256 claimableAmount, uint8 winTier);

```

```solidity
File: src/interfaces/ILotterySetup.sol

154:     function drawScheduledAt(uint128 drawId) external view returns (uint256 time);

159:     function ticketRegistrationDeadline(uint128 drawId) external view returns (uint256 time);

```

```solidity
File: src/interfaces/IRNSource.sol

26:     event RequestedRandomNumber(address indexed consumer, uint256 indexed requestId);

31:     event RequestFulfilled(uint256 indexed requestId, uint256 indexed randomNumber);

```

```solidity
File: src/interfaces/IRNSourceController.sol

40:     event Retry(IRNSource indexed failedSource, uint256 indexed numberOfFailedAttempts);

59:     function failedSequentialAttempts() external view returns (uint256 numberOfAttempts);

62:     function maxFailedAttemptsReachedAt() external view returns (uint256 numberOfAttempts);

65:     function maxFailedAttempts() external view returns (uint256 numberOfAttempts);

```

```solidity
File: src/interfaces/IReferralSystem.sol

23:     event ClaimedReferralReward(uint128 indexed drawId, address indexed user, uint256 indexed claimedAmount);

30:         uint128 indexed drawId, uint256 indexed referrerRewardForDraw, uint256 indexed playerRewardForDraw

36:     function rewardsToReferrersPerDraw(uint256 drawId) external view returns (uint256 reffererRewards);

55:     function totalTicketsForReferrersPerDraw(uint128 drawId) external view returns (uint256 totalNumberOfTickets);

59:     function referrerRewardPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);

63:     function playerRewardsPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);

68:     function claimReferralReward(uint128[] memory drawIds) external returns (uint256 claimedReward);

73:     function minimumEligibleReferrals(uint128 drawId) external view returns (uint256 minimumEligibleReferrals);

```

```solidity
File: src/interfaces/IReferralSystemDynamic.sol

35:         returns (uint256 referralRequirements, uint256 factor, ReferralRequirementFactorType factorType);

```

```solidity
File: src/interfaces/ITicket.sol

29:     function ticketsInfo(uint256 ticketId) external view returns (uint128 drawId, uint120 combination, bool claimed);

```

```solidity
File: src/staking/StakedTokenLock.sol

16:     constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {

39:         if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {

```

```solidity
File: src/staking/Staking.sol

48:     function rewardPerToken() public view override returns (uint256 _rewardPerToken) {

54:         uint256 ticketsSoldSinceUpdate = lottery.nextTicketId() - lastUpdateTicketId;

56:             LotteryMath.calculateRewards(lottery.ticketPrice(), ticketsSoldSinceUpdate, LotteryRewardType.STAKING);

61:     function earned(address account) public view override returns (uint256 _earned) {

62:         return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];

108:     function _beforeTokenTransfer(address from, address to, uint256) internal override {

```

```solidity
File: src/staking/interfaces/IStaking.sol

36:     function userRewardPerTokenPaid(address account) external view returns (uint256);

```

### [NC-16] USE UNDERSCORES FOR NUMBER LITERALS

#### Description:

There are occasions where certain numbers have been hardcoded, either in variable or in the code itself. Large numbers can become hard to read.

#### **Proof Of Concept**

```solidity
File: src/PercentageMath.sol

11:     uint256 public constant ONE_PERCENT = 1000;

```

```solidity
File: src/ReferralSystem.sol

129:         return 5000;

```

## Low Issues

|      | Issue                                                                          |
| ---- | :----------------------------------------------------------------------------- |
| L-1  | DOS WITH BLOCK GAS LIMIT                                                       |
| L-2  | USAGE OF PAYABLE.TRANSFER CAN LEAD TO LOSS OF FUNDS                            |
| L-3  | INSUFFICIENT COVERAGE                                                          |
| L-4  | LACK OF INPUT VALIDATION                                                       |
| L-5  | LOSS OF PRECISION DUE TO ROUNDING                                              |
| L-6  | UNIFY RETURN CRITERIA                                                          |
| L-7  | THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS |
| L-8  | A SINGLE POINT OF FAILURE                                                      |
| L-9  | USE OF `BLOCK.TIMESTAMP`                                                       |
| L-10 | UNSAFE CAST                                                                    |
| L-11 | USE `_SAFEMINT` INSTEAD OF `_MINT`                                             |
| L-12 | USE `SAFETRANSFER` INSTEAD OF `TRANSFER`                                       |

### [L-1] MINE - DoS with block gas limit

#### Description:

Programming patterns such as looping over arrays of unknown size may lead to DoS when the gas cost of execution exceeds the block gas limit.

Reference: https://swcregistry.io/docs/SWC-128

This loops could drain all user gas and revert.

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

111:         uint128[] calldata drawIds,
125:        for (uint256 i = 0; i < drawIds.length; ++i) {

```

```solidity
File: src/ReferralSystem.sol

30:         uint256[] memory _rewardsToReferrersPerDraw
35:        for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {

```

### [L-2] USAGE OF PAYABLE.TRANSFER CAN LEAD TO LOSS OF FUNDS

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

The funds that are to be sent can be lost. The issues with `transfer()` are outlined here:
https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

#### **Proof Of Concept**

```solidity
File: src/staking/StakedTokenLock.sol

47:         stakedToken.transfer(msg.sender, amount);

55:         rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));

```

#### Recommended Mitigation Steps:

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60

### [L-3] INSUFFICIENT COVERAGE

#### Description:

The test coverage rate of the project is below 10%. Testing all functions is best practice in terms of security criteria.

#### **Proof Of Concept**

```
| File                                                  | % Lines          | % Statements     | % Branches      | % Funcs         |
|-------------------------------------------------------|------------------|------------------|-----------------|-----------------|
| src/Lottery.sol                                       | 94.20% (65/69)   | 95.06% (77/81)   | 88.89% (16/18)  | 80.00% (12/15)  |
| src/LotteryMath.sol                                   | 14.29% (2/14)    | 15.79% (3/19)    | 0.00% (0/2)     | 20.00% (1/5)    |
| src/LotterySetup.sol                                  | 71.43% (20/28)   | 65.71% (23/35)   | 68.75% (11/16)  | 50.00% (3/6)    |
| src/LotteryToken.sol                                  | 100.00% (3/3)    | 100.00% (3/3)    | 100.00% (2/2)   | 100.00% (1/1)   |
| src/PercentageMath.sol                                | 0.00% (0/2)      | 0.00% (0/2)      | 100.00% (0/0)   | 0.00% (0/2)     |
| src/RNSourceBase.sol                                  | 86.67% (13/15)   | 87.50% (14/16)   | 75.00% (6/8)    | 100.00% (2/2)   |
| src/RNSourceController.sol                            | 94.74% (36/38)   | 95.00% (38/40)   | 94.44% (17/18)  | 100.00% (6/6)   |
| src/ReferralSystem.sol                                | 39.53% (17/43)   | 41.67% (20/48)   | 25.00% (6/24)   | 71.43% (5/7)    |
| src/Ticket.sol                                        | 100.00% (4/4)    | 100.00% (4/4)    | 100.00% (0/0)   | 100.00% (2/2)   |
| src/TicketUtils.sol                                   | 100.00% (24/24)  | 100.00% (37/37)  | 75.00% (3/4)    | 100.00% (3/3)   |
| src/VRFv2RNSource.sol                                 | 100.00% (4/4)    | 100.00% (4/4)    | 100.00% (2/2)   | 100.00% (2/2)   |
| src/staking/StakedTokenLock.sol                       | 100.00% (10/10)  | 100.00% (10/10)  | 100.00% (4/4)   | 100.00% (3/3)   |
| src/staking/Staking.sol                               | 100.00% (36/36)  | 100.00% (39/39)  | 91.67% (11/12)  | 87.50% (7/8)    |

```

### [L-4] LACK OF INPUT VALIDATION

#### Description:

For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

285:     function mintNativeTokens(address mintTo, uint256 amount) internal override {

```

```solidity
File: src/LotteryToken.sol

22:     function mint(address account, uint256 amount) external override {

```

```solidity
File: src/staking/StakedTokenLock.sol

24:     function deposit(uint256 amount) external override onlyOwner {

37:     function withdraw(uint256 amount) external override onlyOwner {

```

```solidity
File: src/staking/Staking.sol

67:     function stake(uint256 amount) external override {

79:     function withdraw(uint256 amount) public override {

```

### [L-5] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: src/LotteryMath.sol

84:             bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);

```

```solidity
File: src/PercentageMath.sol

18:         return number * percentage / PERCENTAGE_BASE;

23:         return number * int256(percentage) / int256(PERCENTAGE_BASE);

```

### [L-6] UNIFY RETURN CRITERIA

#### Description:

In LotterySetup.sol, ILottery.sol, IReferralSystem.sol, IStakedTokenLock.sol and IStaking.sol files, sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: src/LotterySetup.sol

120:     function fixedReward(uint8 winTier) public view override returns (uint256 amount) {

152:     function drawScheduledAt(uint128 drawId) public view override returns (uint256 time) {

156:     function ticketRegistrationDeadline(uint128 drawId) public view override returns (uint256 time) {

160:     function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {

164:     function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {

```

```solidity
File: src/interfaces/ILottery.sol

92:     function stakingRewardRecipient() external view returns (address rewardRecipient);

95:     function lastDrawFinalTicketId() external view returns (uint256 ticketId);

98:     function drawExecutionInProgress() external view returns (bool);

104:     function winAmount(uint128 drawId, uint8 winTier) external view returns (uint256 amount);

109:     function currentRewardSize(uint8 winTier) external view returns (uint256 rewardSize);

113:     function ticketsSold(uint128 drawId) external view returns (uint256 sold);

116:     function currentNetProfit() external view returns (int256 netProfit);

120:     function unclaimedRewards(LotteryRewardType rewardType) external view returns (uint256 rewards);

123:     function currentDraw() external view returns (uint128 drawId);

128:     function winningTicket(uint128 drawId) external view returns (uint120 winningCombination);

147:         returns (uint256[] memory ticketIds);

152:     function claimRewards(LotteryRewardType rewardType) external returns (uint256 claimedAmount);

159:     function claimWinningTickets(uint256[] calldata ticketIds) external returns (uint256 claimedAmount);

165:     function claimable(uint256 ticketId) external view returns (uint256 claimableAmount, uint8 winTier);

```

```solidity
File: src/interfaces/IReferralSystem.sol

36:     function rewardsToReferrersPerDraw(uint256 drawId) external view returns (uint256 reffererRewards);

39:     function playerRewardFirstDraw() external view returns (uint256);

43:     function playerRewardDecreasePerDraw() external view returns (uint256);

51:         returns (uint128 referrerTicketCount, uint128 playerTicketCount);

55:     function totalTicketsForReferrersPerDraw(uint128 drawId) external view returns (uint256 totalNumberOfTickets);

59:     function referrerRewardPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);

63:     function playerRewardsPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);

68:     function claimReferralReward(uint128[] memory drawIds) external returns (uint256 claimedReward);

73:     function minimumEligibleReferrals(uint128 drawId) external view returns (uint256 minimumEligibleReferrals);

```

```solidity
File: src/staking/interfaces/IStakedTokenLock.sol

15:     function stakedToken() external view returns (IStaking);

18:     function rewardsToken() external view returns (IERC20);

22:     function depositDeadline() external view returns (uint256);

25:     function lockDuration() external view returns (uint256);

28:     function depositedBalance() external view returns (uint256);

```

```solidity
File: src/staking/interfaces/IStaking.sol

30:     function rewardPerTokenStored() external view returns (uint256);

33:     function lastUpdateTicketId() external view returns (uint256);

36:     function userRewardPerTokenPaid(address account) external view returns (uint256);

39:     function rewards(address account) external view returns (uint256);

42:     function lottery() external view returns (ILottery _lottery);

45:     function rewardPerToken() external view returns (uint256 _rewardPerToken);

50:     function earned(address account) external view returns (uint256 _earned);

53:     function rewardsToken() external view returns (IERC20);

56:     function stakingToken() external view returns (IERC20);

```

### [L-7] THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS

#### Description:

If a pair gets created and after a while one of the tokens gets self-destroyed (maybe because of a bug) then `safeTransfer` would still succeed. It’s probably a good idea to check if the contract still exists by checking the bytecode length.

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

107:         nativeToken.safeTransfer(msg.sender, ILotteryToken(address(nativeToken)).INITIAL_SUPPLY());

156:         rewardToken.safeTransfer(beneficiary, claimedAmount);

175:         rewardToken.safeTransfer(msg.sender, claimedAmount);

```

```solidity
File: src/staking/Staking.sol

86:         stakingToken.safeTransfer(msg.sender, amount);

98:             rewardsToken.safeTransfer(msg.sender, reward);

```

### [L-8] A SINGLE POINT OF FAILURE

#### Description:

The owner role has a single point of failure and `onlyOwner` can use critical a few functions.

Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

#### **Proof Of Concept**

```solidity
File: src/RNSourceController.sol

77:     function initSource(IRNSource rnSource) external override onlyOwner {

89:     function swapSource(IRNSource newSource) external override onlyOwner {

```

```solidity
File: src/staking/StakedTokenLock.sol

24:     function deposit(uint256 amount) external override onlyOwner {

37:     function withdraw(uint256 amount) external override onlyOwner {

```

#### Recommended Mitigation Steps:

Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

Also detail them in documentation and NatSpec comments.

### [L-9] USE OF `BLOCK.TIMESTAMP`

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as `block.timestamp` can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of `block.timestamp`, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers can't rely on the preciseness of the provided timestamp.

Reference: https://swcregistry.io/docs/SWC-116
Reference: https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/timestamp-dependence.md

#### **Proof Of Concept**

```solidity
File: src/Lottery.sol

135:         if (block.timestamp < drawScheduledAt(currentDraw)) {

164:             if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {

```

```solidity
File: src/LotterySetup.sol

74:         if (initialPotDeadline < (block.timestamp + lotterySetupParams.drawSchedule.drawPeriod)) {

114:         if (block.timestamp > ticketRegistrationDeadline(drawId)) {

137:         if (block.timestamp <= initialPotDeadline) {

```

```solidity
File: src/RNSourceController.sol

64:         if (block.timestamp - lastRequestTimestamp <= maxRequestDelay) {

70:             maxFailedAttemptsReachedAt = block.timestamp;

94:         bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;

107:         lastRequestTimestamp = block.timestamp;

```

```solidity
File: src/staking/StakedTokenLock.sol

26:         if (block.timestamp > depositDeadline) {

39:         if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {

39:         if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {

```

### [L-10] UNSAFE CAST

#### Description:

Keep in mind that the version of solidity used, despite being greater than 0.8, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

#### **Proof Of Concept**

```solidity
File: src/LotterySetup.sol

170:             uint16 reward = uint16(rewards[winTier] / divisor);

```

```solidity
File: src/ReferralSystem.sol

69:             unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);

71:         unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);

```

```solidity
File: src/TicketUtils.sol

58:             numbers[i] = uint8(randomNumber % currentSelectionCount);

96:                 winTier += uint8(intersection & uint120(1));

```

#### Recommended Mitigation Steps:

Use a safeCast from Open Zeppelin or increase the type length.

### [L-11] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

```solidity
File: src/LotteryToken.sol

19:         _mint(msg.sender, INITIAL_SUPPLY);

26:         _mint(account, amount);

```

```solidity
File: src/Ticket.sol

26:         _mint(to, ticketId);

```

```solidity
File: src/staking/Staking.sol

74:         _mint(msg.sender, amount);

```

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`

### [L-12] USE `SAFETRANSFER` INSTEAD OF `TRANSFER`

#### Description:

It is good to add a `require()` statement that checks the return value of token transfers or to use something like OpenZeppelin’s `safeTransfer`/`safeTransferFrom` unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)‘s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert.

#### **Proof Of Concept**

```solidity
File: src/staking/StakedTokenLock.sol

47:         stakedToken.transfer(msg.sender, amount);

55:         rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));

```

#### Recommended Mitigation Steps:

Consider using `safeTransfer`/`safeTransferFrom` or `require()` consistently.
