## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol

```solidity
// events coming after functions.
61:    event Staked(address indexed user, uint256 indexed amount);
66:    event Withdrawn(address indexed user, uint256 indexed amount);
71:    event RewardPaid(address indexed user, uint256 indexed reward);
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```solidity
// place this external function before all the internal ones
76:    function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {

// place this private function after all the internal ones
136:    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol

```solidity
// place these internal functions after all the external ones
38:    function requestRandomNumber() internal {
58:    function receiveRandomNumber(uint256 randomNumber) internal virtual;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```solidity
// visibility functions are all mixed throughout the code, consider ordering them like this: public, external, internal, private. Within a grouping, place the view and pure functions last.
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol

```solidity
// external functions coming before public ones
67:    function stake(uint256 amount) external override {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStakedTokenLock.sol

```solidity
32:    function deposit(uint256 amount) external;
36:    function withdraw(uint256 amount) external;
40:    function getReward() external;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol

```solidity
// non-view and non-pure functions should come before all the view/pure ones
68:    function claimReferralReward(uint128[] memory drawIds) external returns (uint256 claimedReward);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol

```solidity
// non-view and non-pure functions should come before all the view/pure ones
78:    function onRandomNumberFulfilled(uint256 randomNumber) external;
81:    function initSource(IRNSource rnSource) external;
85:    function retry() external;
90:    function swapSource(IRNSource newSource) external;

```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotteryToken.sol

```solidity
// non-view and non-pure functions should come before all the view/pure ones
22:    function mint(address account, uint256 amount) external;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol

```solidity
// non-view and non-pure functions should come before all the view/pure ones
162:    function finalizeInitialPotRaise() external;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol

```solidity
// non-view and non-pure functions should come before all the view/pure ones
140:    function buyTickets(
152:    function claimRewards(LotteryRewardType rewardType) external returns (uint256 claimedAmount);
159:    function claimWinningTickets(uint256[] calldata ticketIds) external returns (uint256 claimedAmount);
168:    function executeDraw() external;
```

---

### natSpec missing [3]

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol

```solidity
9:    contract VRFv2RNSource is IVRFv2RNSource, RNSourceBase, VRFV2WrapperConsumerBase {
13:    constructor(
//@return missing
28:    function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {

32:    function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol

```solidity
19:    function markAsClaimed(uint256 ticketId) internal {
23:    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```solidity
27:    constructor(
74:    function mintNativeTokens(address mintTo, uint256 amount) internal virtual;
76:    function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {
111:    function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
136:    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
156:    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
161:    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol

```solidity
46:    function onRandomNumberFulfilled(uint256 randomNumber) external override {
58:    function receiveRandomNumber(uint256 randomNumber) internal virtual;
60:    function retry() external override {
77:    function initSource(IRNSource rnSource) external override onlyOwner {
89:    function swapSource(IRNSource newSource) external override onlyOwner {
106:    function requestRandomNumberFromSource() private {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol

```solidity
7:    abstract contract RNSourceBase is IRNSource {
11:    constructor(address _authorizedConsumer) {
48:    function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol

```solidity
22:    function mint(address account, uint256 amount) external override {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```solidity
104:    modifier requireJackpotInitialized() {
112:    modifier beforeTicketRegistrationDeadline(uint128 drawId) {
120:    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
132:    function finalizeInitialPotRaise() external override {
152:    function drawScheduledAt(uint128 drawId) public view override returns (uint256 time) {
156:    function ticketRegistrationDeadline(uint128 drawId) public view override returns (uint256 time) {
160:    function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {
164:    function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```solidity
110:    function buyTickets(
133:    function executeDraw() external override whenNotExecutingDraw {
144:    function unclaimedRewards(LotteryRewardType rewardType) external view override returns (uint256 rewards) {
151:    function claimRewards(LotteryRewardType rewardType) external override returns (uint256 claimedAmount) {
159:    function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier) {
170:    function claimWinningTickets(uint256[] calldata ticketIds) external override returns (uint256 claimedAmount) {
234:    function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
238:    function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {
249:    function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {
259:    function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {
271:    function returnUnclaimedJackpotToThePot() private {
279:    function requireFinishedDraw(uint128 drawId) internal view override {
285:    function mintNativeTokens(address mintTo, uint256 amount) internal override {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol

```solidity
11:   contract Staking is IStaking, ERC20 {
22:    constructor(
48:    function rewardPerToken() public view override returns (uint256 _rewardPerToken) {
61:    function earned(address account) public view override returns (uint256 _earned) {
67:    function stake(uint256 amount) external override {
79:    function withdraw(uint256 amount) public override {
91:    function getReward() public override {
103:    function exit() external override {
108:    function _beforeTokenTransfer(address from, address to, uint256) internal override {
118:    function _updateReward(address account) internal {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

```solidity
9:    contract StakedTokenLock is IStakedTokenLock, Ownable2Step {
16:    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
24:    function deposit(uint256 amount) external override onlyOwner {
37:    function withdraw(uint256 amount) external override onlyOwner {
50:    function getReward() external override {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol

```solidity
// private and internal `functions` should preppend with `underline`
28:    function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {
32:    function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol

```solidity
// private and internal `functions` should preppend with `underline`
19:    function markAsClaimed(uint256 ticketId) internal {
23:    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```solidity
// private and internal `functions` should preppend with `underline`
52:    function referralRegisterTickets(
74:    function mintNativeTokens(address mintTo, uint256 amount) internal virtual;
87:    function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {
111:    function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
134:    function requireFinishedDraw(uint128 drawId) internal view virtual;
136:    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
156:    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
161:    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol

```solidity
// private and internal `functions` should preppend with `underline`
106:    function requestRandomNumberFromSource() private {
38:    function requestRandomNumber() internal {
58:    function receiveRandomNumber(uint256 randomNumber) internal virtual;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol

```solidity
// private and internal `functions` should preppend with `underline`
8:    address internal immutable authorizedConsumer;
9:    mapping(uint256 => RandomnessRequest) internal requests;

//  private and internal `functions` should preppend with `underline`
33:    function fulfill(uint256 requestId, uint256 randomNumber) internal {
48:    function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol

```solidity
// private and internal `functions` should be prefixed with `underline`
17:    function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
22:    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```solidity
// private and internal `variables` should preppend with `underline`
34:    uint256 private immutable nonJackpotFixedRewards;
26:    uint256 internal immutable firstDrawSchedule;

// private and internal `functions` should preppend with `underline`
160:    function _baseJackpot(uint256 _initialPot) internal view returns (uint256) {
164:    function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```solidity
//  private and internal `variables` should preppend with `underline`
25:    uint256 private claimedStakingRewardAtTicketId;
26:    mapping(address => uint256) private frontendDueTicketSales;
27:    mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;

// private and internal `functions` should preppend with `underline`
203:    function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {
279:    function requireFinishedDraw(uint128 drawId) internal view override {
285:    function mintNativeTokens(address mintTo, uint256 amount) internal override {
181:    function registerTicket(
238:    function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {
249:    function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {
259:    function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {
271:    function returnUnclaimedJackpotToThePot() private {
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol

```solidity
// different version from the rest of the repo
3:    pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

```solidity
// different version from the rest of the repo
3:  pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol

```solidity
// different version from the rest of the repo
3:  pragma solidity ^0.8.7;
```