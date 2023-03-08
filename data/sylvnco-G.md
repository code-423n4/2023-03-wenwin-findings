# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use Calldata instead of Memory | 2 |

### <a name="GAS-1"></a>[GAS-1] Use calldata instead of memory for external function
Use calldata instead of memory for external function parameters. This should saves around 100 gas as it's a non modifiable storage (unlike memory). 

*Instances (1)*:
```solidity
File: src/ReferralSystem.sol

76:     function claimReferralReward(uint256[] memory drawIds) external override returns (uint256 claimedReward)

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L76)

```solidity
File: src/interfaces/IReferralSystem.sol

76:     function claimReferralReward(uint256[] memory drawIds) external returns (uint256 claimedReward);

```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/IReferralSystem.sol#L68)


