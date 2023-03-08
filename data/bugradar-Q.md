## General recommendations: 
- Move logic closer together. A lot of single-use modifiers and functions, spread across different contracts.
- (Too) Complex inheritance. Could be simplified/split into different contracts. Unwanted (& unnoticed) shadowing is a real risk in the current codebase imo.

### Unnecessary Imports: 
- LotteryToken.sol; L7: „import "src/LotteryMath.sol“; https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol

### Unnecessary Overrides: 
- LotteryToken.sol; L12, L14, L22: https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryToken.sol#L12

### Unnecessary/Bad Practice Modifiers: Single use modifiers lead to logic far from the function declaration. Makes it hard to read
- Lottery.sol; L44, L51, L60, L69:  Modifiers only used once. Logic should be moved to function declaration. https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L44
- LotterySetup.sol; L104, L112: Modifiers only used once. Logic should be moved to function declaration. https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L104

### Shadowing: especially with complex inheritance like in the WenWin codebase shadowing is a potential origin  of unexpected behavior. 
- Lottery.sol; L86-88: playerRewardFirstDraw param shadows inherited ReferralSystem.sol playerRewardFirstDraw state variable. Same for playerRewardDecreasePerDraw and rewardsToReferrersPerDraw. https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L86
IRNSourceController.sol; L56: return value „(IRNSource source)“ shadows the actual function name. https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/IRNSourceController.sol#L56
IReferralSystem.sol: return value „(uint256 minimumEligibleReferrals)“ shadows the actual function name. https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/IReferralSystem.sol#L73

### Unnecessarily dispersed logic: 
- LotterySetup.sol: moving the logic into Lottery.sol would make the contract a lot easier to review. https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol
- LotterySetup.sol; L120: function only used in Lottery.sol. Should be moved there. 

### Unused Interfaces: 
- IReferralSystemDynamic.sol: seems to be completely unused in the codebase. https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystemDynamic.sol

### Unnecessary code: 
- StakedTokenLock.sol; L17: unnecessary „ _transferOwnership(msg.sender)“. Owner is by default msg.sender.  https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L17

### Use more comments for easier entry into the codebase: 
- LotteryToken.sol; L18: Should have a comment that "Lottery.sol" will be the owner. Helps with understanding. https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryToken.sol#L18

### Pragmas should be locked: 
- StakedTokenLock.sol; L3: https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L3
- VRFv2RNSource.sol; L3: https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/VRFv2RNSource.sol#L3
- IVRFv2RNSource.sol; L3: https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/IVRFv2RNSource.sol#L3

