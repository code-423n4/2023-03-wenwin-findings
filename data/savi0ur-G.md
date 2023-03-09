# [G-01] ++i/i++ should be unchecked{ ++i } / unchecked{ i++ } when its not posssible for them to overflow, as its in the case when used in for loops/while loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L125

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L172

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L211

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L169

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L35

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L77

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L28

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L57

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L65

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L68

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L95


# [G-02] Cache storage values in memory to minimize SLOADS

The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L266

# [G-03] Avoid unnecessary read of array length in for/while loops can save gas

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L121-L130

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L35

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L32

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L77

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L33-L34


