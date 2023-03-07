# QA Report

## Use fixed compiler version

A floating pragma risks a different compiler version being used in production vs testing, which poses security risks.

Instances: 2
- [src/VRFv2RNSource.sol#L3](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L3)
- [src/staking/StakedTokenLock.sol#L3](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L3)

<br>

## Implementing `renounceOwnership` is dangerous

Typically, the contract's owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The OpenZeppelin's Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design.

Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

It is recommended to either reimplement the function to disable it, or to clearly specify if it is part of the contract design.


Instances: 3
- [src/RNSourceController.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L5)
- [src/interfaces/IRNSourceController.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L5)
- [src/staking/StakedTokenLock.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L5)

<br>

## Remove unused imports

Improves readability and saves gas.

Instances: 5
- [src/LotteryToken.sol#L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L7)
- [src/LotteryMath.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L5)
- [src/interfaces/ILotterySetup.sol#L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L6)
- [src/interfaces/IRNSourceController.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L5)
- [src/interfaces/IReferralSystem.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L5)

<br>

## Missing `address(0)` checks in constructor/initialize

Failing to check for invalid parameters on deployment may result in an erroneous input and require an expensive redeployment.

Instances: 3
- [src/RNSourceBase.sol#L11](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L11)
- [src/VRFv2RNSource.sol#L13](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L13)
- [src/staking/StakedTokenLock.sol#L16](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L16)

<br>
