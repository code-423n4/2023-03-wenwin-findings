# Non-Critical Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[N-01]|Include all relevant parameters in events|3|
|[N-02]|`finalizeInitialPotRaise` is vulnerable to frontrunning|1|
|[N-03]|Missing argument check in `swapSource`|1|
|[N-04]|Use fixed compiler version|2|
|[N-05]|Implementing `renounceOwnership` is dangerous|3|
|[N-06]|Remove unused imports|5|
|[N-07]|Missing `address(0)` checks in constructor/initialize|3|

&nbsp;
Total issues: 7
Total instances: 18

&nbsp;
# Non-Critical Issues

## [N-01] Include all relevant parameters in events

Include an `oldSource` parameter in `RNSourceController`'s `SourceSet` event to provide full information about the change:
- [src/interfaces/IRNSourceController.sol#L44](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L44)
- [src/RNSourceController.sol#L86](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L86)
- [src/RNSourceController.sol#L102](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L102)

&nbsp;
## [N-02] `finalizeInitialPotRaise` is vulnerable to frontrunning

Should the contract meet the `minInitialPot` threshold before the team has finished funding the contract completely or before they are ready for the lottery to begin, anyone can call the `finalizeInitialPotRaise` function to begin the lottery.

Make sure to either fully fund the contract in one transaction, or add access control to `finalizeInitialPotRaise`.

Instances: 1
- [src/LotterySetup.sol#L132](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L132)

&nbsp;
## [N-03] Missing argument check in `swapSource`

A check should be added to `swapSource` to ensure that `newSource` is not equal to the old `source` value.

- [src/RNSourceController.sol#L89](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L89)


## [N-04] Use fixed compiler version

A floating pragma risks a different compiler version being used in production vs testing, which poses security risks.

Instances: 2
- [src/VRFv2RNSource.sol#L3](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L3)
- [src/staking/StakedTokenLock.sol#L3](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L3)

&nbsp;
## [N-05] Implementing `renounceOwnership` is dangerous

Typically, the contract's owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The OpenZeppelin's Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design.

Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

It is recommended to either reimplement the function to disable it, or to clearly specify if it is part of the contract design.


Instances: 3
- [src/RNSourceController.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L5)
- [src/interfaces/IRNSourceController.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L5)
- [src/staking/StakedTokenLock.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L5)

&nbsp;
## [N-06] Remove unused imports

Improves readability and saves gas.

Instances: 5
- [src/LotteryToken.sol#L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L7)
- [src/LotteryMath.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L5)
- [src/interfaces/ILotterySetup.sol#L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L6)
- [src/interfaces/IRNSourceController.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L5)
- [src/interfaces/IReferralSystem.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L5)

&nbsp;
## [N-07] Missing `address(0)` checks in constructor/initialize

Failing to check for invalid parameters on deployment may result in an erroneous input and require an expensive redeployment.

Instances: 3
- [src/RNSourceBase.sol#L11](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L11)
- [src/VRFv2RNSource.sol#L13](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L13)
- [src/staking/StakedTokenLock.sol#L16](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L16)

&nbsp;