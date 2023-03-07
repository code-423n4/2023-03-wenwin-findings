# QA REPORT

---

### Summary of low risk issues


| Number     | Issue details                                    | Instances |
| ------------ | -------------------------------------------------- | :---------: |
| [L-1](#L1) | Use of`block.timestamp`                          |    11    |
| [L-2](#L2) | `require()` should be used instead of `assert()` |     2     |
| [L-3](#L3) | Lock pragmas to specific compiler version.       |     3     |

*Total: 3 issues.*

### Summary of non-critical issues


| Number       | Issue details                                                                         | Instances |
| -------------- | --------------------------------------------------------------------------------------- | :---------: |
| [NC-1](#NC1) | For modern and more readable code, update import usages.                              |    53    |

*Total: 1 issue.*

---

### Low Risk Issues

### <a id=L1>[L-1]</a> Use of `block.timestamp`

##### Description

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the *[Entropy Illusion](https://hacken.io/discover/most-common-smart-contract-vulnerabilities/#Entropy_Illusion)* for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

##### Recommendation

Use oracle instead of `block.timestamp`

##### *Instances (11):*

File: [2023-03-wenwin/src/Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L135 )

```solidity
135: if (block.timestamp < drawScheduledAt(currentDraw)) {
164: if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {
```

File: [2023-03-wenwin/src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L74 )

```solidity
74: if (initialPotDeadline < (block.timestamp + lotterySetupParams.drawSchedule.drawPeriod)) {
114: if (block.timestamp > ticketRegistrationDeadline(drawId)) {
137: if (block.timestamp <= initialPotDeadline) {
```

File: [2023-03-wenwin/src/RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L64 )

```solidity
64: if (block.timestamp - lastRequestTimestamp <= maxRequestDelay) {
70: maxFailedAttemptsReachedAt = block.timestamp;
94: bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;
107: lastRequestTimestamp = block.timestamp;
```

File: [2023-03-wenwin/src/staking/StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L26 )

```solidity
26: if (block.timestamp > depositDeadline) {
39: if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {
```


### <a id=L2>[L-2]</a> `require()` should be used instead of `assert()`

##### Description

Prior to solidity version 0.8.0, hitting an assert consumes the remainder of the transaction's available gas rather than returning it, as `require()`/`revert()` do. `assert()` should be avoided even past solidity version 0.8.0 as its [documentation](https://docs.soliditylang.org/en/v0.8.14/control-structures.html#panic-via-assert-and-error-via-require) states that The assert function creates an error of type Panic(uint256). ... Properly functioning code should never create a Panic, not even on invalid external input. 

##### Recommendation

Consider changing from `assert()` to `require()`

##### *Instances (2):*

File: [2023-03-wenwin/src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147 )

```solidity
147: assert(initialPot > 0);
```

File: [2023-03-wenwin/src/TicketUtils.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99 )

```solidity
99: assert((winTier <= selectionSize) && (intersection == uint256(0)));
```

### <a id=L3>[L-3]</a> Lock pragmas to specific compiler version.

##### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

##### Recommendation

Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version. [solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

##### *Instances (3):*

File: [2023-03-wenwin/src/VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L3 )

```solidity
3: pragma solidity ^0.8.7;
```

File: [2023-03-wenwin/src/interfaces/IVRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L3 )

```solidity
3: pragma solidity ^0.8.7;
```

File: [2023-03-wenwin/src/staking/StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L3 )

```solidity
3: pragma solidity ^0.8.17;
```


### Non-critical Issues



### <a id=NC1>[NC-1]</a> For modern and more readable code, update import usages.

##### Description

Solidity code is cleaner in the following way: On the principle that clearer code is better code, you should import the things you want to use. Specific imports with curly braces allow us to apply this rule better. Check out this [article](https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a)

##### Recommendation

Import like this: `import {contract1 , contract2} from "filename.sol";`

##### *Instances (53):*

File: [2023-03-wenwin/src/Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
6: import "@openzeppelin/contracts/utils/math/Math.sol";
7: import "src/ReferralSystem.sol";
8: import "src/RNSourceController.sol";
9: import "src/staking/Staking.sol";
10: import "src/LotterySetup.sol";
11: import "src/TicketUtils.sol";
```

File: [2023-03-wenwin/src/LotteryMath.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L5 )

```solidity
5: import "src/interfaces/ILottery.sol";
6: import "src/PercentageMath.sol";
```

File: [2023-03-wenwin/src/LotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/utils/math/Math.sol";
6: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
7: import "src/PercentageMath.sol";
8: import "src/LotteryToken.sol";
9: import "src/interfaces/ILotterySetup.sol";
10: import "src/Ticket.sol";
```

File: [2023-03-wenwin/src/LotteryToken.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
6: import "src/interfaces/ILotteryToken.sol";
7: import "src/LotteryMath.sol";
```

File: [2023-03-wenwin/src/RNSourceBase.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L5 )

```solidity
5: import "src/interfaces/IRNSource.sol";
```

File: [2023-03-wenwin/src/RNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/access/Ownable2Step.sol";
6: import "src/interfaces/IRNSource.sol";
7: import "src/interfaces/IRNSourceController.sol";
```

File: [2023-03-wenwin/src/ReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/utils/math/Math.sol";
6: import "src/interfaces/IReferralSystem.sol";
7: import "src/PercentageMath.sol";
```

File: [2023-03-wenwin/src/Ticket.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
6: import "src/interfaces/ITicket.sol";
```

File: [2023-03-wenwin/src/VRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L5 )

```solidity
5: import "@chainlink/contracts/src/v0.8/VRFV2WrapperConsumerBase.sol";
6: import "src/interfaces/IVRFv2RNSource.sol";
7: import "src/RNSourceBase.sol";
```

File: [2023-03-wenwin/src/interfaces/ILottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol#L5 )

```solidity
5: import "src/interfaces/ILotterySetup.sol";
6: import "src/interfaces/IRNSourceController.sol";
7: import "src/interfaces/ITicket.sol";
8: import "src/interfaces/IReferralSystem.sol";
```

File: [2023-03-wenwin/src/interfaces/ILotterySetup.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
6: import "src/interfaces/ITicket.sol";
```

File: [2023-03-wenwin/src/interfaces/ILotteryToken.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotteryToken.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

File: [2023-03-wenwin/src/interfaces/IRNSourceController.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/access/Ownable2Step.sol";
6: import "src/interfaces/IRNSource.sol";
```

File: [2023-03-wenwin/src/interfaces/IReferralSystem.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L5 )

```solidity
5: import "src/interfaces/ILotteryToken.sol";
```

File: [2023-03-wenwin/src/interfaces/ITicket.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ITicket.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/token/ERC721/IERC721.sol";
```

File: [2023-03-wenwin/src/interfaces/IVRFv2RNSource.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L5 )

```solidity
5: import "src/interfaces/IRNSource.sol";
```

File: [2023-03-wenwin/src/staking/StakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/access/Ownable2Step.sol";
6: import "src/staking/interfaces/IStakedTokenLock.sol";
7: import "src/staking/interfaces/IStaking.sol";
```

File: [2023-03-wenwin/src/staking/Staking.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
6: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
7: import "src/interfaces/ILottery.sol";
8: import "src/LotteryMath.sol";
9: import "src/staking/interfaces/IStaking.sol";
```

File: [2023-03-wenwin/src/staking/interfaces/IStakedTokenLock.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStakedTokenLock.sol#L5 )

```solidity
5: import "src/staking/interfaces/IStaking.sol";
```

File: [2023-03-wenwin/src/staking/interfaces/IStaking.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol#L5 )

```solidity
5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
6: import "src/interfaces/ILottery.sol";
```

