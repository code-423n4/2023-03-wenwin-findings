LOW

## 1. `_safeMint()` should be used rather than `_mint()` wherever possible

`_mint()` is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. Both OpenZeppelin and solmate have versions of this function so that NFTs aren’t lost if they’re minted to contracts that cannot transfer them back out.

```
19       _mint(msg.sender, INITIAL_SUPPLY);
26       _mint(account, amount);
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol
```
74       _mint(msg.sender, amount);
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol
```
26       _mint(to, ticketId);
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol


## 2. An unbounded loop on array can lead to DoS

The interface and the function should require a start index and a maximum lenght, so that the index composition can be fetched in batches without running out of gas. If there are thousands of index components, the function may revert.

```
30	uint256[] memory _rewardsToReferrersPerDraw
35	for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
...
76	function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) {
77		for (uint256 counter = 0; counter < drawIds.length; ++counter) {
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol
```
124	ticketIds = new uint256[](tickets.length);
125	for (uint256 i = 0; i < drawIds.length; ++i) {
126		ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
...
170	function claimWinningTickets(uint256[] calldata ticketIds) external override returns (uint256 claimedAmount) {
171		uint256 totalTickets = ticketIds.length;
172		for (uint256 i = 0; i < totalTickets; ++i) {
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol


# NON-CRITICAL


## 1. Use a more recent version of Solidity

- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value
- Use a solidity version of at least 0.8.12 to get `string.concat()` to be used instead of `abi.encodePacked(<str>,<str>)`
- Use a solidity version of at least 0.8.13 to get the ability to use `using for` with a list of free functions
- Use a solidity version of 0.8.15 where conditions necessary for inlining are relaxed. It shows significant dicrease in the bytecode size (and thus reduced deployment cost)
- Use a solidity version of 0.8.17 to prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero; More efficient overflow checks for multiplication

```
3        pragma solidity ^0.8.7;
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol


## 2. Unused constructor

The constructor does nothing.

```
17       constructor() ERC20("Wenwin Lottery", "LOT") {
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol


## 3. For modern and more readable code; update import usages

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

Recommendation
`import {contract1 , contract2} from "filename.sol";`

```
All contracts
```


## 4. For functions, follow Solidity standard naming conventions

The above codes don’t follow Solidity’s standard naming convention,

- internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

- public and external functions : only mixedCase


```
All contracts - internal functions do not start with an underscore
```

## 5. `require()` should be used instead of `assert()`

Prior to solidity version 0.8.0, hitting an assert consumes the remainder of the transaction’s available gas rather than returning it, as `require()`/`revert()` do. `assert()` should be avoided even past solidity version 0.8.0 as its [documentation](https://docs.soliditylang.org/en/v0.8.14/control-structures.html#panic-via-assert-and-error-via-require) states that “The assert function creates an error of type Panic(uint256). … Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix”.

```
147      assert(initialPot > 0);
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol
```
99       assert((winTier <= selectionSize) && (intersection == uint256(0)));
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol


## 6. Avoid nested `if` statements

For better readability and analysis it is better to avoid nested `if` blocks if possible.

```
60	if (referrer != address(0)) {
61		uint256 minimumEligible = minimumEligibleReferrals[currentDraw];
62		if (unclaimedTickets[currentDraw][referrer].referrerTicketCount + numberOfTickets >= minimumEligible) {
63			if (unclaimedTickets[currentDraw][referrer].referrerTicketCount < minimumEligible) {
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol
```
161	if (!ticketInfo.claimed) {
162		uint120 _winningTicket = winningTicket[ticketInfo.drawId];
163		winTier = TicketUtils.ticketWinTier(ticketInfo.combination, _winningTicket, selectionSize, selectionMax);
164		if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {
165			claimableAmount = winAmount[ticketInfo.drawId][winTier];
166		}
167	}
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol


## 6. `Function writing` that does not comply with the `Solidity Style Guide`


Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last


https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol

