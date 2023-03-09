
# Gas Optimization Report

This report focuses on Wenwin contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Wenwin Protocol are categorised into 07 main areas; with further multiple instances in each of the category.


# Summary

[G-01] 0perator assignment is more gas efficient than compound assignment (16 Instances)
[G-02] Require is more gas efficient than assert (02 Instances)
[G-03] Immutable has more gas efficiency than constant (04 Instances)
[G-04] Multiple mappings can be combined into a single one (12 Instances)
[G-05] Wastage of deployed gas when return is not present for returns function (11 Instances)
[G 06] Declare Public library as internal library (05 Instances)
[G-07] Public visibility consumes more gas as compared to external in functions (02 Instances)
 
# [G-01] 0perator assignment is more gas efficient than compound assignment (16 Instances)
Compound assignment operators (+= / -=) are more expensive in terms of gas consumption and needs to be avoided.

Operator assignments (a = a + b / a - b) are preferable in terms of gas optimization.

Link to the Code:
1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L43
3.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L129
4.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L173
5.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275
6.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L64
7.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L67
8.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L69
9.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L71
10.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L78
11.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L147
12.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L29
13.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L96
14.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L55
15.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L64
16.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L84

# [G-02] Require is more gas efficient than assert (02 Instances)

Assert() and require() functions are similar in nature, regarding context of handling error and undone any changes made.

However, assert() consumes all the gas during the process of reverting change while require()refunds any gas remaining that was paid.


Link to the Code:
1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99

# [G-03] Immutable has more gas efficiency than constant (04 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost

Link to the Code:

1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L12
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L36
3.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L19
4.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L20

# [G-04] Multiple mappings can be combine into a single one (12 Instances)

When multiple mappings are used in same function, it’s better to combined them into a single mapping using a struct.

Combined mapping reduces storage slot per mapping and also are cheaper in terms of associated stack operations calculation carried out.

Link to the Code:

1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L19
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L20
3.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L26
4.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L27
5.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L36
6.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L37
7.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L39
8.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L17
9.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L19
10.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L21
11.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L23
12.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L25

# [G-05] Wastage of deployed gas when return is not present for returns function (11 Instances)
Wastage of gas during deployment; when return is absent for named variable when function returns.

Link to the Code:

1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L28
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L152
3.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L156
4.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L164
5.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L144
6.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L74
7.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L35
8.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L62
9.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L73
10.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L96
11.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L119

# [G 06] Declare Public library as internal library (05 Instances)
External call to a public library function is very expensive for reference checks this article. When library has only internal functions it can be used as internal library.

Link to the code:

1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L7
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L8
3.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L7
4.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L11
5.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L7


# [G-07] Public visibility consumes more gas as compared to external in functions (02 Instances)
Functions with public visibility, if not called within the contract needed to be changed external.

Link to the Code:
1.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120
2.	https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L234
