## Summary

### Low Risk Issues
|      | Issue                                                                            | Instances |
| ---- | :------------------------------------------------------------------------------- | :-------: |
| L-01 | Upgrade open zeppelin contract dependency                                        |     1     |
| L-02 | lockDuration in StakedTokenLock should be adapted to documentation               |     1     |
| L-03 | callbackGasLimit in VRFv2RNSource is immutable                                   |     1     |
| L-04 | Align VRFv2RNSource.requestConfirmations and RNSourceController._maxRequestDelay |     1     |
| L-05 | nonJackpotFixedRewards in LotterySetup has some limitations                      |     2     |

**Total: 6 instances over 5 issues**


### Non-critical Issues
|       | Issue                                                        | Instances |
| ----- | :----------------------------------------------------------- | :-------: |
| NC-01 | Defined return value not used                                |    10     |
| NC-02 | Unnecessarily imports                                        |     5     |
| NC-03 | Use specific imports instead of just a global import         |    20     |
| NC-04 | Consider a not so strict versioning for your main interfaces |    11     |
| NC-05 | Solidity pragma versioning                                   |     2     |

**Total: 48 instances over 5 issues**


### Informational
|      | Issue                                          | Instances |
| ---- | :--------------------------------------------- | :-------: |
| I-01 | IReferralSystemDynamic is unused               |     1     |
| I-02 | Missing NatSpec @inheritdoc in implementations |    10     |
| I-03 | Missing documentation                          |    22     |

**Total: 33 instances over 3 issues**

---

## Low Risk Issues

### L-01 Upgrade open zeppelin contract dependency
An OZ version with a known vulnerabilities for batchMinting ERC721 tokens is used. Even if the ERC721Consecutive is not used, you should consider Updating to the fixed Version 0.8.2.

[Release Note](https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.8.2)

[Security Advisories](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-878m-3g6q-594q)

### L-02 lockDuration in StakedTokenLock should be adapted to documentation
The documentation says that the Team token allocation will be locked for 1 year.
https://docs.wenwin.com/wenwin-lottery/protocol-architecture/token

`20% (200,000,000 tokens) is allocated to the team, which is pre-staked and locked for 1 year before the Initial Sale begins`

Currently the `lockDuration` can be set on deployment, to be aligned with the documentation you should consider a constant of 365 days for it.

### L-03 callbackGasLimit in VRFv2RNSource is immutable
Currently the callbackGasLimit in VRFv2RNSource is immutable and doesn't check if at least a minimum amount is set.
If the callbackGasLimit is not high enough, it will break the lottery as it is not possible to get any random number and the contracts will be stale until `RNSourceController.swapSource` can be used to update the `source`.

You should consider a minimum check for `callbackGasLimit` in the constructor.

### L-04 Align VRFv2RNSource.requestConfirmations and RNSourceController._maxRequestDelay
Currently it is possible, that the variable RNSourceController.maxRequestDelay is smaller than the time VRFv2RNSource.requestConfirmations will be.
If the maxRequestDelay is for example 5 minutes and the value for requestConfirmations is 300 (2 sec block time = 10 minutes) it's possible to use `RNSourceController.retry()` before Chainlink will fulfill the random number request.

Also you should add a minimum check for `maxRequestDelay` in the constructor.

### L-05 nonJackpotFixedRewards in LotterySetup has some limitations
The `packFixedRewards` function in LotterySetup has some limitations that will prevent a Lottery where a user can get back a small amount if no match and will break depending on the decimals of the rewardToken.

```solidity
File: src/LotterySetup.sol
165:         if (rewards.length != (selectionSize) || rewards[0] != 0) {
```
Prevents a Lottery where a user could get a small amount back if he didn't have a match.

```solidity
File: src/LotterySetup.sol
170:         uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1);
```
Will break if decimals for the rewardToken = 0.

### L-xx 

---

## Non-critical Issues 

### NC-01 Defined return value not used
If you're not using the variable name in the function declaration you should remove it.

```solidity
File: src/staking/Staking.sol
48:     function rewardPerToken() public view override returns (uint256 _rewardPerToken) {

File: src/staking/Staking.sol
61:     function earned(address account) public view override returns (uint256 _earned) {

File: src/Lottery.sol
234:     function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {

File: src/Lottery.sol
238:     function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize) {

File: src/LotterySetup.sol
120:     function fixedReward(uint8 winTier) public view override returns (uint256 amount) {

File: src/PercentageMath.sol
17:     function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {

File: src/PercentageMath.sol
22:     function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {

File: src/ReferralSystem.sol
156:     function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

File: src/ReferralSystem.sol
161:     function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

File: src/TicketUtils.sol
17:     function isValidTicket(
18:         uint256 ticket,
19:         uint8 selectionSize,
20:         uint8 selectionMax
21:     )
22:         internal
23:         pure
24:         returns (bool isValid)
```

### NC-02 Unnecessarily imports
Some contracts have unnecessary imports that are never used and can be removed.

```solidity
File: src/interfaces/ILotterySetup.sol
6: import "src/interfaces/ITicket.sol";

File: src/interfaces/IReferralSystem.sol
5: import "src/interfaces/ILotteryToken.sol";

File: src/interfaces/IRNSourceController.sol
5: import "@openzeppelin/contracts/access/Ownable2Step.sol";

File: src/Lottery.sol
6: import "@openzeppelin/contracts/utils/math/Math.sol";

File: src/LotteryToken.sol
7: import "src/LotteryMath.sol";
```

### NC-03 Use specific imports instead of just a global import
For a better better developer experience it's better to use specific imports instead of just a global import. This helps to have a better overview what is realy needed and helps to have a clearer view of the contract.

```solidity
File: src/interfaces/ILottery.sol

File: src/interfaces/ILotterySetup.sol

File: src/interfaces/ILotteryToken.sol

File: src/interfaces/IReferralSystem.sol

File: src/interfaces/IRNSourceController.sol

File: src/interfaces/ITicket.sol

File: src/interfaces/IVRFv2RNSource.sol

File: src/staking/interfaces/IStakedTokenLock.sol

File: src/staking/interfaces/IStaking.sol

File: src/staking/StakedTokenLock.sol

File: src/staking/Staking.sol

File: src/Lottery.sol

File: src/LotteryMath.sol

File: src/LotterySetup.sol

File: src/LotteryToken.sol

File: src/ReferralSystem.sol

File: src/RNSourceBase.sol

File: src/RNSourceController.sol

File: src/Ticket.sol

File: src/VRFv2RNSource.sol
```

### NC-04 Consider a not so strict versioning for your main interfaces
At the moment your interfaces have a strict version for 0.8.19. Consider a more open Version like ^0.8.0 so other projects can import your original interfaces and dont need to rewrite or change them if they use another version than 0.8.19.

```solidity
File: src/interfaces/ILottery.sol

File: src/interfaces/ILotterySetup.sol

File: src/interfaces/ILotteryToken.sol

File: src/interfaces/IReferralSystem.sol

File: src/interfaces/IReferralSystemDynamic.sol

File: src/interfaces/IRNSource.sol

File: src/interfaces/IRNSourceController.sol

File: src/interfaces/ITicket.sol

File: src/interfaces/IVRFv2RNSource.sol

File: src/staking/interfaces/IStakedTokenLock.sol

File: src/staking/interfaces/IStaking.sol
```

### NC-05 Solidity pragma versioning
It's better to have a fixed version for your implementation contracts. Currently you have defined ^0.8.17 and ^0.8.7 in two contracts. You should use the fixed Version 0.8.19 for them like in the others.

For your interfaces you could consider a ^0.8.0 version.

```solidity
File: src/staking/StakedTokenLock.sol
3: pragma solidity ^0.8.17;

File: src/VRFv2RNSource.sol
3: pragma solidity ^0.8.7;
```

---

## Informational

### I-01 IReferralSystemDynamic is unused
The Interface IReferralSystemDynamic is currently unused and not implemented by any contract.

### I-02 Missing NatSpec @inheritdoc in implementations
You should consider adding the NatSpec @inheritdoc for the functions implementing an Interface definition.

```solidity
File: src/staking/StakedTokenLock.sol

File: src/staking/Staking.sol

File: src/Lottery.sol

File: src/LotterySetup.sol

File: src/LotteryToken.sol

File: src/ReferralSystem.sol

File: src/RNSourceBase.sol

File: src/RNSourceController.sol

File: src/Ticket.sol

File: src/VRFv2RNSource.sol
```

### I-03 Missing documentation
The contracts are well documented and clear to read. Still it misses some documentation that could be added for a even better developer experience. Functions missing @param (or wrong param mentioned) / @return / @notice or is missing complete NatSpec.

```solidity
File: src/interfaces/ILotteryToken.sol
13:     function INITIAL_SUPPLY() external view returns (uint256 initialSupply);

File: src/interfaces/IReferralSystem.sol
39:     function playerRewardFirstDraw() external view returns (uint256);

File: src/interfaces/IReferralSystem.sol
43:     function playerRewardDecreasePerDraw() external view returns (uint256);

File: src/interfaces/IReferralSystem.sol
45:     function unclaimedTickets(

File: src/interfaces/IReferralSystem.sol
59:     function referrerRewardPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);

File: src/interfaces/IReferralSystem.sol
63:     function playerRewardsPerDrawForOneTicket(uint128 drawId) external view returns (uint256 rewardsPerDraw);

File: src/interfaces/IReferralSystemDynamic.sol
32:     function referralRequirements(uint256 index)

File: src/interfaces/IRNSourceController.sol
56:     function source() external view returns (IRNSource source);

File: src/interfaces/IRNSourceController.sol
59:     function failedSequentialAttempts() external view returns (uint256 numberOfAttempts);

File: src/interfaces/IRNSourceController.sol
62:     function maxFailedAttemptsReachedAt() external view returns (uint256 numberOfAttempts);

File: src/interfaces/IRNSourceController.sol
81:     function initSource(IRNSource rnSource) external;

File: src/staking/interfaces/IStakedTokenLock.sol
15:     function stakedToken() external view returns (IStaking);

File: src/staking/interfaces/IStakedTokenLock.sol
18:     function rewardsToken() external view returns (IERC20);

File: src/staking/interfaces/IStakedTokenLock.sol
22:     function depositDeadline() external view returns (uint256);

File: src/staking/interfaces/IStakedTokenLock.sol
25:     function lockDuration() external view returns (uint256);

File: src/staking/interfaces/IStakedTokenLock.sol
28:     function depositedBalance() external view returns (uint256);

File: src/staking/interfaces/IStaking.sol
30:     function rewardPerTokenStored() external view returns (uint256);

File: src/staking/interfaces/IStaking.sol
33:     function lastUpdateTicketId() external view returns (uint256);

File: src/staking/interfaces/IStaking.sol
36:     function userRewardPerTokenPaid(address account) external view returns (uint256);

File: src/staking/interfaces/IStaking.sol
39:     function rewards(address account) external view returns (uint256);

File: src/staking/interfaces/IStaking.sol
53:     function rewardsToken() external view returns (IERC20);

File: src/staking/interfaces/IStaking.sol
56:     function stakingToken() external view returns (IERC20);
```