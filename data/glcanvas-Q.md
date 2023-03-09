# [Q-1]

Rename constant ```MAX_MAX_FAILED_ATTEMPTS``` to ```MAX_FAILED_ATTEMPTS```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L19

# [Q-2]

Move out variable out of immutable scope 

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L21 

variable initialPot is placed between immutable variables, due to improve readability place this variable after ```nonJackpotFixedRewards```

# [Q-3]

Use constants instead of numbers

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L51
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L55
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L61
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L58
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L62




  