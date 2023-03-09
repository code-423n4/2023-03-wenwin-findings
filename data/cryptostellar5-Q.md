
### 01 REENTRANCY - CHECKS-EFFECTS-INTERACTION PATTERN NOT BEING FOLLOWED

*Number of Instances Identified: 1*

In the `stake()` function the `_mint()` is being called after the `stakingToken.safeTransferFrom()` external interaction 

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L67-L77

```
function stake(uint256 amount) external override {
        // _updateReward is not needed here as it's handled by _beforeTokenTransfer
        if (amount == 0) {
            revert ZeroAmountInput();
        }

        stakingToken.safeTransferFrom(msg.sender, address(this), amount);
        _mint(msg.sender, amount);

        emit Staked(msg.sender, amount);
    }
```

### 02 ADDING A RETURN STATEMENT WHEN THE FUNCTION DEFINES A NAMED RETURN VARIABLE, IS REDUNDANT

*Number of Instances Identified: 11*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
234: function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize)
238: function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
120: function fixedReward(uint8 winTier) public view override returns (uint256 amount)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol

```
17: function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result)
22: function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
111-115: function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
        internal
        view
        virtual
        returns (uint256 minimumEligible)
156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards)
161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```
17-24: function isValidTicket(
        uint256 ticket,
        uint8 selectionSize,
        uint8 selectionMax
    )
        internal
        pure
        returns (bool isValid)

```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol

```
48: function rewardPerToken() public view override returns (uint256 _rewardPerToken)
61: function earned(address account) public view override returns (uint256 _earned)
```

### 03 FLOATING PRAGMA VERSION SHOULD NOT BE USED

*Number of Instances Identified: 3*

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

[https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)


#### Proof of Concept

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol

```
pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol

```
pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

```
pragma solidity ^0.8.17;
```



### 04 REQUIRE() SHOULD BE USED INSTEAD OF ASSERT()

*Number of Instances Identified: 2*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
147: assert(initialPot > 0);
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol

```
99: assert((winTier <= selectionSize) && (intersection == uint256(0)));
```


### 05 CONSTANTS SHOULD BE DEFINED RATHER THAN USING MAGIC NUMBERS

*Number of Instances Identified: 3*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
48: if (lotterySetupParams.selectionSize == 0)
51: if (lotterySetupParams.selectionMax >= 120)
55: lotterySetupParams.expectedPayout < lotterySetupParams.ticketPrice / 100
```

### 06 USE LATEST SOLIDITY VERSION

*Number of Instances Identified: 3*

When deploying contracts, you should use the latest released version of Solidity. Apart from exceptional cases, only the latest version receives [security fixes](https://github.com/ethereum/solidity/security/policy#supported-versions). Furthermore, breaking changes as well as new features are introduced regularly.
Latest Version is 0.8.19

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol

```
pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol

```
pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

```
pragma solidity ^0.8.17;
```



### 07 FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

*Number of Instances Identified: 14*

### Description

Our Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

### Recommendation

`import {contract1 , contract2} from "filename.sol";`

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

```
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/utils/math/Math.sol";
import "src/ReferralSystem.sol";
import "src/RNSourceController.sol";
import "src/staking/Staking.sol";
import "src/LotterySetup.sol";
import "src/TicketUtils.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol

```
import "src/interfaces/ILottery.sol";
import "src/PercentageMath.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol

```
import "@openzeppelin/contracts/utils/math/Math.sol"; 
import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
import "src/PercentageMath.sol";
import "src/LotteryToken.sol";
import "src/interfaces/ILotterySetup.sol";
import "src/Ticket.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol

```
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "src/interfaces/ILotteryToken.sol";
import "src/LotteryMath.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol

```
import "@openzeppelin/contracts/utils/math/Math.sol";
import "src/interfaces/IReferralSystem.sol";
import "src/PercentageMath.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol

```
import "src/interfaces/IRNSource.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol

```
import "@openzeppelin/contracts/access/Ownable2Step.sol";
import "src/interfaces/IRNSource.sol";
import "src/interfaces/IRNSourceController.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol

```
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "src/interfaces/ITicket.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol

```
import "@chainlink/contracts/src/v0.8/VRFV2WrapperConsumerBase.sol";
import "src/interfaces/IVRFv2RNSource.sol";
import "src/RNSourceBase.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol

```
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "src/interfaces/ITicket.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol

```
import "src/interfaces/ILotterySetup.sol";
import "src/interfaces/IRNSourceController.sol";
import "src/interfaces/ITicket.sol";
import "src/interfaces/IReferralSystem.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotteryToken.sol

```
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol

```
import "src/interfaces/ILotteryToken.sol";
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol

```
import "@openzeppelin/contracts/access/Ownable2Step.sol";
import "src/interfaces/IRNSource.sol";
```