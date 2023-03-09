# Gas Optimization

# Summary

| Number | Optimization Details                                                                     | Context |
| :----: | :--------------------------------------------------------------------------------------- | :-----: |
| [G-01] | internal functions only called once can be inlined to save gas                           |   16    |
| [G-02] | Functions guaranteed to revert when called by normal users can be marked payable         |    4    |
| [G-03] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES                       |    3    |
| [G-04] | Setting the constructor to payable                                                       |   10    |
| [G-05] | Not Using The Named Return Variables when A function Returns, Waste Deployment Gas       |   86    |
| [G-06] | USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS |    2    |
| [G-07] | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD          |    1    |
| [G-08] | SHOULD USE ARGUMENTS INSTEAD OF STATE VARIABLE                                           |    1    |
| [G-09] | Duplicated require()/if() checks should be refactored to a modifier or function          |    1    |
| [G-10] | Use constants instead of type(uintx).max                                                 |    1    |
| [G-11] | Use nested if and, avoid multiple check combinations                                     |    2    |
| [G-12] | Use assembly to write address storage values                                             |    2    |
| [G-13] | A modifier used only once and not being inherited should be inlined to save gas          |    4    |
| [G-14] | modifier not used in the contract, Waste Deployment Gas                                  |    2    |

## [G-01] **internal** functions only called once can be **inlined** to save gas

### Summary

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

### Details

If function called once in the contract will be inline to save gas

There are **16** instances of this issue.

### Github Permalinks

```solidity
File: /src/VRFv2RNSource.sol

28:    function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {

32:    function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol

```solidity
File: /src/Lottery.sol

203:     function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {

279:     function requireFinishedDraw(uint128 drawId) internal view override {

285:     function mintNativeTokens(address mintTo, uint256 amount) internal override {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```solidity
File: /src/Ticket.sol

19:     function markAsClaimed(uint256 ticketId) internal {

23:     function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol

```solidity
File: /src/ReferralSystem.sol

52    function referralRegisterTickets(
53        uint128 currentDraw,
54        address referrer,
55        address player,
56        uint256 numberOfTickets
57    )
58:        internal
59    {

87:   function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```solidity
File: /src/PercentageMath.sol

17:     function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {

22:     function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol

```solidity
File: /src/TicketUtils.sol

17    function isValidTicket(
18        uint256 ticket,
19        uint8 selectionSize,
20        uint8 selectionMax
21    )
22:        internal
23        pure
24        returns (bool isValid)
25    {

43    function reconstructTicket(
44        uint256 randomNumber,
45        uint8 selectionSize,
46        uint8 selectionMax
47    )
48:        internal
49        pure
50        returns (uint120 ticket)
51    {

83    function ticketWinTier(
84        uint120 ticket,
85        uint120 winningTicket,
86        uint8 selectionSize,
87        uint8 selectionMax
88    )
89:        internal
90        pure
91        returns (uint8 winTier)
92    {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```solidity
File: /src/LotteryMath.sol

35    function calculateNewProfit(
36        int256 oldProfit,
37        uint256 ticketsSold,
38        uint256 ticketPrice,
39        bool jackpotWon,
40        uint256 fixedJackpotSize,
41        uint256 expectedPayout
42    )
43:        internal
44        pure
45        returns (int256 newProfit)
46    {

119:     function calculateRewards(
120        uint256 ticketPrice,
121        uint256 ticketsSold,
122        LotteryRewardType rewardType
123    )
124:        internal
125        pure
126        returns (uint256 dueRewards)
127    {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol

### Mitigation

instead of internal use inline.

## [G-2] Functions guaranteed to **revert** when called by normal users can be marked **payable**

### Summary

These are missed from **4naly3er**

https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae#gas-9-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable

### Details

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

There are **4** instances of this issue.

### Github Permalinks

```solidity
File: /src/staking/StakedTokenLock.sol

24:     function deposit(uint256 amount) external override onlyOwner {

37:     function withdraw(uint256 amount) external override onlyOwner {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

```solidity
File: /Ethos-Core/contracts/BorrowerOperations.sol

77:     function initSource(IRNSource rnSource) external override onlyOwner {

89:     function swapSource(IRNSource newSource) external override onlyOwner {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol

### Mitigation

Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

## [G-3] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES

### Summary

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

### Details

There are **3** instances of this issue.

### Github Permalinks

```solidity
File: /src/staking/StakedTokenLock.sol

//uint256 public override depositedBalance;

30:         depositedBalance += amount;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30

```solidity
File: /src/Lottery.sol

//int256 public override currentNetProfit;

275:            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275

```solidity
File: /src/staking/StakedTokenLock.sol

//uint256 public override depositedBalance;

82:        depositedBalance -= amount;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L43

### Mitigation

Avoid compound assignment operator in **state variables**

## [G-4] Setting the constructor to payable

### Summary

Saves ~13 gas per instance

### Details

There are **10** instance of this issue

### Github Permalinks

```solidity
File: /src/LotteryToken.sol

17:     constructor() ERC20("Wenwin Lottery", "LOT") {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L17

```solidity
File: /src/VRFv2RNSource.sol

13     constructor(
14        address _authorizedConsumer,
15        address _linkAddress,
16        address _wrapperAddress,
17        uint16 _requestConfirmations,
18        uint32 _callbackGasLimit
19    )
20        RNSourceBase(_authorizedConsumer)
21        VRFV2WrapperConsumerBase(_linkAddress, _wrapperAddress)
22    {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L13

```solidity
File: /src/staking/StakedTokenLock.sol

16:     constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L16

```solidity
File: /src/staking/Staking.sol

22    constructor(
23        ILottery _lottery,
24        IERC20 _rewardsToken,
25        IERC20 _stakingToken,
26        string memory name,
27        string memory symbol
28    )
29        ERC20(name, symbol)
30    {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L22

```solidity
File: /src/LotterySetup.sol

41:     constructor(LotterySetupParams memory lotterySetupParams)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L41

```solidity
File: /src/Lottery.sol

84    constructor(
85        LotterySetupParams memory lotterySetupParams,
86        uint256 playerRewardFirstDraw,
87        uint256 playerRewardDecreasePerDraw,
88        uint256[] memory rewardsToReferrersPerDraw,
89        uint256 maxRNFailedAttempts,
90        uint256 maxRNRequestDelay
91    )
92        Ticket()
93        LotterySetup(lotterySetupParams)
94        ReferralSystem(playerRewardFirstDraw, playerRewardDecreasePerDraw, rewardsToReferrersPerDraw)
95        RNSourceController(maxRNFailedAttempts, maxRNRequestDelay)
96    {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L84

```solidity
File: /src/Ticket.sol

17     constructor() ERC721("Wenwin Lottery Ticket", "WLT") { }
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L17

```solidity
File: /src/RNSourceBase.sol

11     constructor(address _authorizedConsumer) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L11

```solidity
File: /src/RNSourceController.sol

26     constructor(uint256 _maxFailedAttempts, uint256 _maxRequestDelay) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L26

```solidity
File: /src/ReferralSystem.sol

27     constructor(
28        uint256 _playerRewardFirstDraw,
29        uint256 _playerRewardDecreasePerDraw,
30        uint256[] memory _rewardsToReferrersPerDraw
31    ) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L27

### Mitigation

Setting the constructor to payable

## [G-5] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

### Summary

Do not use return variables when a function returns

### Details

There are **86** instances of this issue.

### Github Permalinks

```solidity
File: /src/VRFv2RNSource.sol

28:     function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L28

```solidity
File: /src/staking/Staking.sol

48:    function rewardPerToken() public view override returns (uint256 _rewardPerToken) {

61:    function earned(address account) public view override returns (uint256 _earned) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol

```solidity
File: /src/LotterySetup.sol

120:    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {

152:    function drawScheduledAt(uint128 drawId) public view override returns (uint256 time) {

156:    function ticketRegistrationDeadline(uint128 drawId) public view override returns (uint256 time) {

164:    function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```solidity
File: /src/Lottery.sol

119:         returns (uint256[] memory ticketIds)

144:    function unclaimedRewards(LotteryRewardType rewardType) external view override returns (uint256 rewards) {

151:    function claimRewards(LotteryRewardType rewardType) external override returns (uint256 claimedAmount) {

159:    function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier) {

170:    function claimWinningTickets(uint256[] calldata ticketIds) external override returns (uint256 claimedAmount) {

190:        returns (uint256 ticketId)

234:    function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {

238:    function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {

249:    function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {

259:    function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```solidity
File: /src/Ticket.sol

23:    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L23

```solidity
File: /src/RNSourceBase.sol

48:     function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L48

```solidity
File: /src/ReferralSystem.sol

76:    function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {

115:        returns (uint256 minimumEligible)

136:    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {

156:    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

161:    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```solidity
File: /src/PercentageMath.sol

17:    function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {

22:    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L17

```solidity
File: /src/TicketUtils.sol

24:        returns (bool isValid)

50:        returns (uint120 ticket)

91:        returns (uint8 winTier)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L24

```solidity
File: /src/LotteryMath.sol

45:        returns (int256 newProfit)

62:     function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot) {

80:        returns (uint256 bonusMulti)

106:        returns (uint256 reward)

126:        returns (uint256 dueRewards)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol

```solidity
File: /src/interfaces/IVRFv2RNSource.sol

14:     function callbackGasLimit() external returns (uint32 gasLimit);

17:    function requestConfirmations() external returns (uint16 minConfirmations);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol

```solidity
File: /src/interfaces/ILotteryToken.sol

13:     function INITIAL_SUPPLY() external view returns (uint256 initialSupply);

16:    function owner() external view returns (address _owner);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotteryToken.sol

```solidity
File: /src/interfaces/ITicket.sol

22:    function nextTicketId() external view returns (uint256 nextId);

29:    function ticketsInfo(uint256 ticketId) external view returns (uint128 drawId, uint120 combination, bool claimed);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ITicket.sol

```solidity
File: /src/interfaces/IReferralSystemDynamic.sol

35:        returns (uint256 referralRequirements, uint256 factor, ReferralRequirementFactorType factorType);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystemDynamic.sol#L35

```solidity
File: /src/staking/interfaces/IStaking.sol

42:    function lottery() external view returns (ILottery _lottery);

45:    function rewardPerToken() external view returns (uint256 _rewardPerToken);

50:    function earned(address account) external view returns (uint256 _earned);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol

```solidity
File: /src/interfaces/IReferralSystem.sol

36:    function rewardsToReferrersPerDraw(uint256 drawId) external view returns (uint256 reffererRewards);

51:        returns (uint128 referrerTicketCount, uint128 playerTicketCount);

55:    function totalTicketsForReferrersPerDraw(uint128 drawId) external view returns (uint256 totalNumberOfTickets);

59:    function referrerRewardPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);

63:    function playerRewardsPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);

68:    function claimReferralReward(uint128[] memory drawIds) external returns (uint256 claimedReward);

73:    function minimumEligibleReferrals(uint128 drawId) external view returns (uint256 minimumEligibleReferrals);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol

```solidity
File: /src/interfaces/IRNSourceController.sol

56:    function source() external view returns (IRNSource source);

59:    function failedSequentialAttempts() external view returns (uint256 numberOfAttempts);

62:    function maxFailedAttemptsReachedAt() external view returns (uint256 numberOfAttempts);

65:    function maxFailedAttempts() external view returns (uint256 numberOfAttempts);

68:    function lastRequestTimestamp() external view returns (uint256 timestamp);

71:    function lastRequestFulfilled() external view returns (bool isFulfilled);

74:    function maxRequestDelay() external view returns (uint256 delay);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol

```solidity
File: /src/interfaces/ILottery.sol

92:    function stakingRewardRecipient() external view returns (address rewardRecipient);

95:    function lastDrawFinalTicketId() external view returns (uint256 ticketId);

104:    function winAmount(uint128 drawId, uint8 winTier) external view returns (uint256 amount);

109:    function currentRewardSize(uint8 winTier) external view returns (uint256 rewardSize);

113:    function ticketsSold(uint128 drawId) external view returns (uint256 sold);

116:    function currentNetProfit() external view returns (int256 netProfit);

120:    function unclaimedRewards(LotteryRewardType rewardType) external view returns (uint256 rewards);

123:    function currentDraw() external view returns (uint128 drawId);

128:    function winningTicket(uint128 drawId) external view returns (uint120 winningCombination);

147:        returns (uint256[] memory ticketIds);

152:    function claimRewards(LotteryRewardType rewardType) external returns (uint256 claimedAmount);

159:    function claimWinningTickets(uint256[] calldata ticketIds) external returns (uint256 claimedAmount);

165:    function claimable(uint256 ticketId) external view returns (uint256 claimableAmount, uint8 winTier);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol

```solidity
File: /src/interfaces/ILotterySetup.sol

104:    function minInitialPot() external view returns (uint256 minPot);

107:    function jackpotBound() external view returns (uint256 bound);

112:    function rewardToken() external view returns (IERC20 token);

115:    function nativeToken() external view returns (IERC20 token);

121:    function ticketPrice() external view returns (uint256 price);

125:    function fixedReward(uint8 winTier) external view returns (uint256 amount);

128:    function initialPot() external view returns (uint256 potSize);

132:    function selectionSize() external view returns (uint8 size);

137:    function selectionMax() external view returns (uint8 max);

140:    function expectedPayout() external view returns (uint256 payout);

143:    function drawPeriod() external view returns (uint256 period);

146:    function initialPotDeadline() external view returns (uint256 topUpEndsAt);

149:    function drawCoolDownPeriod() external view returns (uint256 period);

154:    function drawScheduledAt(uint128 drawId) external view returns (uint256 time);

159:    function ticketRegistrationDeadline(uint128 drawId) external view returns (uint256 time);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol

## [G-6] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS

### Details

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

There is **2** instance of this issue.

### Github Permalinks

```solidity
File: /src/ReferralSystem.sol

76:     function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L76

```solidity
File: /src/interfaces/IReferralSystem.sol

68:    function claimReferralReward(uint128[] memory drawIds) external returns (uint256 claimedReward);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L68

## [G-7] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

### Summary

Contracts are allowed to override their parents functions and change the visibility from external to public and can save gas by doing so, public functions not called in the contract itself should be declared externals.

### Details

if a function is override public and not called in the contract should be declared external instead

There are **1** instance of this issue.

### Github Permalinks

```solidity
File: /src/Lottery.sol

234:    function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L234

## [G-8] SHOULD USE ARGUMENTS INSTEAD OF STATE VARIABLE

### Summary

state variables should not used in emit

### Details

There are **1** instance of this issue.

### Github Permalinks

```solidity
File: /src/Lottery.sol

//34:    uint128 public override currentDraw;  state variable

141:         emit StartedExecutingDraw(currentDraw);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L141

## [G-09] Duplicated require()/if() checks should be refactored to a modifier or function

### Summary

### Details

There are **1** instances of this issue

### Github Permalinks

```solidity
File: /src/staking/Staking.sol

69:        if (amount == 0) {
81:        if (amount == 0) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L69

## [G-10] Use constants instead of type(uintx).max

### Summary

type(uintx).max it uses more gas in the distribution process and also for each transaction than constant usage.

### Details

there are **1** instances of this issue

### Github Permalinks

```solidity
File: /src/LotterySetup.sol

126:           uint256 mask = uint256(type(uint16).max) << (winTier * 16);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L126

## [G-11] Use nested if and, avoid multiple check combinations

### Summary

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

### Details

there are **2** instances of this issue

### Github Permalinks

```solidity
File: /src/staking/StakedTokenLock.sol

39:         if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L39

```solidity
File: /src/LotteryMath.sol

83:         if (excessPot > 0 && ticketsSold > 0) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L83

### Recomendation Code

```solidity
File: /src/LotteryMath.sol

-83:    if (excessPot > 0 && ticketsSold > 0) {

+       if(excessPot > 0) {
+          if(ticketsSold > 0){
            }

           ...}
```

## [G-12] Use assembly to write address storage values

### Details

there are **2** instances of this issue

### Github Permalinks

```solidity
File: /src/staking/StakedTokenLock.sol

18:         stakedToken = IStaking(_stakedToken);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L18

```solidity
File: /src/RNSourceBase.sol

12:        authorizedConsumer = _authorizedConsumer;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L12

### Recommendation Code

```solidity
File: /src/RNSourceBase.sol

    constructor(address _authorizedConsumer) {
        assembly {
            sstore(authorizedConsumer.slot,_authorizedConsumer)
        }
    }
```

## [G-13] A modifier used only once and not being inherited should be inlined to save gas

### Summary

### Details

there are **4** instances of this issue

### Github Permalinks

```solidity
File: /src/Lottery.sol

44    modifier requireValidTicket(uint256 ticket) {
45        if (!ticket.isValidTicket(selectionSize, selectionMax)) {
46            revert InvalidTicket();
47        }
48        _;
49    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L44

### The above modifer is only used in the following:

```solidity
File: /src/Lottery.sol

181    function registerTicket(
182        uint128 drawId,
183        uint120 ticket,
184        address frontend,
185        address referrer
186    )
187        private
188        beforeTicketRegistrationDeadline(drawId)
189        requireValidTicket(ticket)
190        returns (uint256 ticketId)
200    {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L181-L196

```solidity
File: /src/Lottery.sol

52    modifier whenNotExecutingDraw() {
53        if (drawExecutionInProgress) {
54            revert DrawAlreadyInProgress();
55        }
56        _;
57    }



60 modifier onlyWhenExecutingDraw() {
61        if (!drawExecutionInProgress) {
62            revert DrawNotInProgress();
63        }
64        _;
65    }



69    modifier onlyTicketOwner(uint256 ticketId) {
70            if (ownerOf(ticketId) != msg.sender) {
71                revert UnauthorizedClaim(ticketId, msg.sender);
72            }
73            _;
74       }
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

## [G-14] A modifier not used in the contract, Waste Deployment Gas

### Details

there are **2** instances of this issue

### Github Permalinks

```solidity
File: /src/LotterySetup.sol

104    modifier requireJackpotInitialized() {
105        // slither-disable-next-line incorrect-equality
106        if (initialPot == 0) {
107            revert JackpotNotInitialized();
108        }
109        _;
110    }



112    modifier beforeTicketRegistrationDeadline(uint128 drawId) {
113        // slither-disable-next-line timestamp
114        if (block.timestamp > ticketRegistrationDeadline(drawId)) {
115           revert TicketRegistrationClosed(drawId);
116        }
117        _;
118    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L104-L118
