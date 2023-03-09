
# Low Risk and Non-Critical Issues

Some of the opportunities identified for improving low severity issues throughout the codebase of Wenwin protocol are categorised into 03 main areas; with further multiple instances in each of the category.


# [L-01] Floating of Pragma (03 Instances)
Locking pragma version ensures contracts are not being deployed on an outdated compiler version.

Link to the code:
1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L3
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L3
3.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L3


# [L 02] Avoid using latest version of Solidity due to unknown bugs (21 Instances)

Precaution should be taken in using solidity latest released version that can impact the project based on reason of unknown bugs.

Link to the code:

1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L3
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L3
3.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L3
4.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L3
5.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L3
6.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L3
7.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L3
8.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L3
9.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L3
10.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L3
11.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L3
12.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotteryToken.sol#L3
13.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ITicket.sol#L3
14.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStakedTokenLock.sol#L3
15.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystemDynamic.sol#L3
16.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSource.sol#L3
17.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol#L3
18.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L3
19.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L3
20.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol#L3
21.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L3



# [L-03] Minting tokens to the zero address should be avoided (04 Instances)

Address(0) check is missing in function, consider applying check to ensure tokens or tickets aren’t minted to the zero address.

Link to the code:
1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L22
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L74
3.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L26
4.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L81


# [N-01] Constructor lacks address(0) check (03 Instances)

Zero-address check should be used in the constructors, to avoid the risk of setting a storage variable as address(0) at deploying time.

Link to the code:
1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L13
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L16
3.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L11

