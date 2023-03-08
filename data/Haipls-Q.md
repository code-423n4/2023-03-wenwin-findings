## Hardcode the parameters causes no future updates
```solidity
VRFv2RNSource.sol
10:    uint16 public immutable override requestConfirmations;
11:    uint32 public immutable override callbackGasLimit;
```
Updating third-party dependencies is a standard practice such as gas price per operation or errors in consensus leading to more block reorganizations
You need to adapt to these situations. And hardcoding parameters related to this is a bad practice.
It is not always possible to redeploy VRFV2RNSource, as it can work in 1/5 cases. But it will greatly affect the experience of using the lottery

## Hardcode the address causes no future updates
```solidity
src/Lottery.sol
29:    address public immutable override stakingRewardRecipient;

src/RNSourceBase.sol
8:    address internal immutable authorizedConsumer;

```
In case the addresses change due to reasons such as updating their versions in the future, addresses coded as constants cannot be updated, so it is recommended to add the update option with the onlyOwner modifier.

## Presence of unused variables
```solidity 
src/Lottery.sol

260:    uint256 winTier; // never used after initialization L261
261:        (claimedAmount, winTier) = this.claimable(ticketId);
```

## SWC-119 Shadowing State Variables
```solidity 
src/Lottery.sol

94:         ReferralSystem(playerRewardFirstDraw, playerRewardDecreasePerDraw, rewardsToReferrersPerDraw)

```

## For functions, follow Solidity standard naming conventions (internal, private variable and function style rule) and ordering
```solidity
src/Ticket.sol:

19:     function markAsClaimed(uint256 ticketId) internal {
23:     function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {

src/Lottery.sol

25:     uint256 private claimedStakingRewardAtTicketId;
26:    mapping(address => uint256) private frontendDueTicketSales;
27:    mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
181:    function registerTicket
203:    function receiveRandomNumber(uint256 randomNumber) internal
238:    function drawRewardSize(uint128 drawId, uint8 winTier) private
249:    function dueTicketsSoldAndReset(address beneficiary) private
259:    function claimWinningTicket(uint256 ticketId) private
271:    function returnUnclaimedJackpotToThePot() private
279:    function requireFinishedDraw(uint128 drawId) internal
285:    function mintNativeTokens(address mintTo, uint256 amount) internal

src/LotterySetup.sol

26:     uint256 internal immutable firstDrawSchedule;
34:     uint256 private immutable nonJackpotFixedRewards;
164:    function packFixedRewards(uint256[] memory rewards)

src/RNSourceBase.sol

33:    function fulfill(uint256 requestId, uint256 randomNumber) internal
48:    function requestRandomnessFromUnderlyingSource()

src/RNSourceController.sol

38:    function requestRandomNumber() internal
58:    function receiveRandomNumber(uint256 randomNumber) internal
106:    function requestRandomNumberFromSource() private {

src/ReferralSystem.sol

52:    function referralRegisterTickets
74:    function mintNativeTokens(address mintTo, uint256 amount) internal virtual;
87:    function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal
111:    function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
        internal
134:    function requireFinishedDraw(uint128 drawId) internal view virtual;
136:    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward)
156:    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards)
161:    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards)
```