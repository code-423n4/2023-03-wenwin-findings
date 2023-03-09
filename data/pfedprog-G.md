## Don't Initialize Variables with Default Value

Uninitialized variables are assigned with the types default value.

Explicitly initializing a variable with it's default value costs unnecessary gas.

  https://github.com/code-423n4/2023-03-wenwin/src/Lottery.sol::125 => for (uint256 i = 0; i < drawIds.length; ++i) {
  https://github.com/code-423n4/2023-03-wenwin/src/Lottery.sol::172 => for (uint256 i = 0; i < totalTickets; ++i) {
  https://github.com/code-423n4/2023-03-wenwin/src/ReferralSystem.sol::35 => for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
  https://github.com/code-423n4/2023-03-wenwin/src/ReferralSystem.sol::77 => for (uint256 counter = 0; counter < drawIds.length; ++counter) {
  https://github.com/code-423n4/2023-03-wenwin/src/TicketUtils.sol::28 => for (uint8 i = 0; i < selectionMax; ++i) {
  https://github.com/code-423n4/2023-03-wenwin/src/TicketUtils.sol::57 => for (uint256 i = 0; i < selectionSize; ++i) {
  https://github.com/code-423n4/2023-03-wenwin/src/TicketUtils.sol::65 => for (uint256 i = 0; i < selectionSize; ++i) {
  https://github.com/code-423n4/2023-03-wenwin/src/TicketUtils.sol::68 => for (uint256 j = 0; j <= currentNumber; ++j) {
  https://github.com/code-423n4/2023-03-wenwin/src/TicketUtils.sol::95 => for (uint8 i = 0; i < selectionMax; ++i) {

## Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.
https://github.com/code-423n4/2023-03-wenwin/src/Lottery.sol::121 => if (drawIds.length != tickets.length) {
  https://github.com/code-423n4/2023-03-wenwin/src/Lottery.sol::122 => revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
  https://github.com/code-423n4/2023-03-wenwin/src/Lottery.sol::124 => ticketIds = new uint256[](tickets.length);
  https://github.com/code-423n4/2023-03-wenwin/src/Lottery.sol::125 => for (uint256 i = 0; i < drawIds.length; ++i) {
  https://github.com/code-423n4/2023-03-wenwin/src/Lottery.sol::128 => referralRegisterTickets(currentDraw, referrer, msg.sender, tickets.length);
  https://github.com/code-423n4/2023-03-wenwin/src/Lottery.sol::129 => frontendDueTicketSales[frontend] += tickets.length;
  https://github.com/code-423n4/2023-03-wenwin/src/Lottery.sol::130 => rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);
  https://github.com/code-423n4/2023-03-wenwin/src/Lottery.sol::171 => uint256 totalTickets = ticketIds.length;
  https://github.com/code-423n4/2023-03-wenwin/src/LotterySetup.sol::165 => if (rewards.length != (selectionSize) || rewards[0] != 0) {
  https://github.com/code-423n4/2023-03-wenwin/src/ReferralSystem.sol::32 => if (_rewardsToReferrersPerDraw.length == 0) {
  https://github.com/code-423n4/2023-03-wenwin/src/ReferralSystem.sol::35 => for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
  https://github.com/code-423n4/2023-03-wenwin/src/ReferralSystem.sol::77 => for (uint256 counter = 0; counter < drawIds.length; ++counter) {
  https://github.com/code-423n4/2023-03-wenwin/src/ReferralSystem.sol::162 => return rewardsToReferrersPerDraw[Math.min(rewardsToReferrersPerDraw.length - 1, drawId)];
  https://github.com/code-423n4/2023-03-wenwin/src/VRFv2RNSource.sol::33 => if (randomWords.length != 1) {
  https://github.com/code-423n4/2023-03-wenwin/src/VRFv2RNSource.sol::34 => revert WrongRandomNumberCountReceived(requestId, randomWords.length);
  https://github.com/code-423n4/2023-03-wenwin/src/interfaces/ILottery.sol::23 => /// @dev Provided arrays of drawIds and Tickets with different length.
  https://github.com/code-423n4/2023-03-wenwin/src/interfaces/ILottery.sol::133 => /// Reverts in case of insufficient `rewardToken`(`tickets.length * ticketPrice`) in `msg.sender`'s account.
  https://github.com/code-423n4/2023-03-wenwin/src/interfaces/ILottery.sol::134 => /// Requires approval to spend `msg.sender`'s `rewardToken` of at least `tickets.length * ticketPrice`
  https://github.com/code-423n4/2023-03-wenwin/src/interfaces/ILottery.sol::136 => /// @param tickets list of uint120 packed tickets. Needs to be of same length as `drawIds`.


## FOR-LOOPS: ++I COSTS LESS GAS COMPARED TO I++

++i costs less gas compared to i++ for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration)

Ticket.mint(address,uint128,uint120) (src/Ticket.sol#23-27) has costly operations inside a loop:
	- ticketId = nextTicketId ++ (src/Ticket.sol#24)
