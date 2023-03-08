## `_safeMint()` should be used rather than `_mint()` wherever possible

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L26](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L26)

### Vulnerability details

`_mint()` is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) has version of this function so that NFTs aren’t lost if they’re minted to contracts that cannot transfer them back out.

```solidity
abstract contract Ticket is ITicket, ERC721 {
    uint256 public override nextTicketId;
    mapping(uint256 => ITicket.TicketInfo) public override ticketsInfo;

    // solhint-disable-next-line no-empty-blocks
    constructor() ERC721("Wenwin Lottery Ticket", "WLT") { }

    function markAsClaimed(uint256 ticketId) internal {
        ticketsInfo[ticketId].claimed = true;
    }

    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
        ticketId = nextTicketId++;
        ticketsInfo[ticketId] = TicketInfo(drawId, combination, false);
        _mint(to, ticketId);
    }
}
```

### Recommendation

Use `_safeMint` instead of `_mint`

## `private` and `internal` functions should start with `_`

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol)

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol)

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol)

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol)

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol)

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol)

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol)

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol)

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol)

It is a convention in Solidity to declare `private` and `internal` functions starting with `_`

```solidity
src/Lottery.sol
205:    function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {
240:    function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {
251:    function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {
261:    function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {
273:    function returnUnclaimedJackpotToThePot() private {
281:    function requireFinishedDraw(uint128 drawId) internal view override {
287:    function mintNativeTokens(address mintTo, uint256 amount) internal override {

src/LotteryMath.sol
62:    function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot) {

src/LotterySetup.sol
164:    function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {

src/PercentageMath.sol
17:    function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
22:    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {

src/RNSourceBase.sol
33:    function fulfill(uint256 requestId, uint256 randomNumber) internal {
48:    function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId);

src/RNSourceController.sol
38:    function requestRandomNumber() internal {
58:    function receiveRandomNumber(uint256 randomNumber) internal virtual;
106:    function requestRandomNumberFromSource() private {

src/ReferralSystem.sol
75:    function mintNativeTokens(address mintTo, uint256 amount) internal virtual;
88:    function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {
136:    function requireFinishedDraw(uint128 drawId) internal view virtual;
139:    function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
159:    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
164:    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

src/Ticket.sol
19:    function markAsClaimed(uint256 ticketId) internal {
23:    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {

src/VRFv2RNSource.sol
28:    function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {
32:    function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {
```

## `assert` should be replaced with `require`

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147)

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99)

The Solidity `assert()` function is meant to assert invariants and `require()` analyses conditions. When condition inside the `assert` is false, it tends to consume all of the remaining gas, but when condition inside `require` is false it refunds all the remaining gas of the transaction `gasLimit`

```solidity
src/LotterySetup.sol
147:  assert(initialPot > 0);

src/TicketUtils.sol
99:  assert((winTier <= selectionSize) && (intersection == uint256(0)));
```

## Time values can be reduced from `uint256` to `uint64` which would be safe for more than 500 years

[https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L53-L60](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L53-L60)

```diff
diff --git a/src/interfaces/ILotterySetup.sol.orig b/src/interfaces/ILotterySetup.sol
index 780d1a3..b7ad39e 100644
--- a/src/interfaces/ILotterySetup.sol.orig
+++ b/src/interfaces/ILotterySetup.sol
@@ -52,11 +52,11 @@ error TicketRegistrationClosed(uint128 drawId);
 /// @dev Lottery draw schedule parameters
 struct LotteryDrawSchedule {
     /// @dev First draw is scheduled to take place at this timestamp
-    uint256 firstDrawScheduledAt;
+    uint64 firstDrawScheduledAt;
     /// @dev Period for running lottery
-    uint256 drawPeriod;
+    uint64 drawPeriod;
     /// @dev Cooldown period when users cannot register tickets for draw anymore
-    uint256 drawCoolDownPeriod;
+    uint64 drawCoolDownPeriod;
 }
```