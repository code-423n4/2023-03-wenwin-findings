# [L-1] The amount of pre-staked tokens should be specified in the constructor of the Lottery Token contract and locked automatically

In white paper of Wenwin, 20% (200,000,000 tokens) is allocated to the team, which is pre-staked and locked for 1 year before the Initial Sale begins. 
https://docs.wenwin.com/wenwin-lottery/protocol-architecture/token

Instead of sending the 200 million LOT tokens to the admin account, they should be created and deposited directly into the locked staking contract. This will enhance the trust of users and investors.
https://github.com/code-423n4/2023-03-wenwin/blob/a260ecba74720c631ab06ecdd67167317b976b60/src/LotteryToken.sol#L17-L20

## Recommended Mitigation Steps

Mint directly the pre-staked amount to staking contract and send staked tokens into the `StakedTokenLock` contract.


# [NC-1] Check address (0) of important input

Instance:

https://github.com/code-423n4/2023-03-wenwin/blob/a260ecba74720c631ab06ecdd67167317b976b60/src/Lottery.sol#L113-L114


# [NC-2] Remove key word `this` in function call

In Solidity, you can call a function in the same contract by its name without using `this.function`. This is the recommended way to write code as it is more concise and clear.

https://github.com/code-423n4/2023-03-wenwin/blob/a260ecba74720c631ab06ecdd67167317b976b60/src/Lottery.sol#L261