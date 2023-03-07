## 1. Use a more recent version of solidity

- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value
- Use a solidity version of at least 0.8.12 to get `string.concat()` to be used instead of `abi.encodePacked(<str>,<str>)`
- Use a solidity version of at least 0.8.13 to get the ability to use `using for` with a list of free functions
- Use a solidity version of 0.8.15 where conditions necessary for inlining are relaxed. It shows significant dicrease in the bytecode size (and thus reduced deployment cost)
- Use a solidity version of 0.8.17 to prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero; More efficient overflow checks for multiplication

```
3        pragma solidity ^0.8.7;
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol


## 2. `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables

```
275      currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol
```
30       depositedBalance += amount;
43       depositedBalance -= amount;
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol


## 3. Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

In almost all contracts we can see examples of usage of uitn128, uint120, uint8 etc. Consider using a larger size then downcast where needed.


## 4. Setting the constructor to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

```
All contracts where constructor exists
```


## 5. Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key’s keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

```
17	mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;
19	mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;
21	mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;
23	mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;
25	mapping(uint128 => uint256) public override minimumEligibleReferrals;
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol
```
27	mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
36	mapping(uint128 => uint120) public override winningTicket;
37	mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
39	mapping(uint128 => uint256) public override ticketsSold;
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol


## 6. Not using the named return variables when a function returns, wastes deployment gas

```
17	function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
18		return number * percentage / PERCENTAGE_BASE;
...
22	function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
23		return number * int256(percentage) / int256(PERCENTAGE_BASE);
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol
```
156	function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
157		uint256 decrease = uint256(drawId) * playerRewardDecreasePerDraw;
158		return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;
...
161	function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
162		return rewardsToReferrersPerDraw[Math.min(rewardsToReferrersPerDraw.length - 1, drawId)];
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol
```
48	function rewardPerToken() public view override returns (uint256 _rewardPerToken) {
58		return rewardPerTokenStored + (unclaimedRewards * 1e18 / _totalSupply);
...
61	function earned(address account) public view override returns (uint256 _earned) {
62		return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol







