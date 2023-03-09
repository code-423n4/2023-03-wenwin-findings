# No `delete` in the code

## Impact
We can easily find that there is no `delete` in the code.

## Proof of Concept
In contract `Lottery`, there are several multiple mappings such as `unclaimedCount` and `winAmount`.
Since they never deleted, the space is getting bigger and bigger, so it can cause the heavy contract.

## Recommended Mitigation Steps
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L271-L277
We can add some deletion activities here. We can delete the 1 year old data.

# Maximum fixed reward is $6553.5 - too small in case of `selectionSize = 16`

Let's think about maximum fixed reward.
We can easily guess that the maximum fixed reward is $6553.5 in current code.

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L164-L176
``` js
    function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {
        if (rewards.length != (selectionSize) || rewards[0] != 0) {
            revert InvalidFixedRewardSetup();
        }
        uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1);
        for (uint8 winTier = 1; winTier < selectionSize; ++winTier) {
            uint16 reward = uint16(rewards[winTier] / divisor);
            if ((rewards[winTier] % divisor) != 0) {
                revert InvalidFixedRewardSetup();
            }
            packed |= uint256(reward) << (winTier * 16);
        }
    }
```

From this code, we can find that each fixed reward is stored as 16 bits with the unit of 0.1 USD.
So the maximum fixed reward is `(2 ^ 16 - 1) * 0.1 = 6553.5`.
This is no problem when `selectionSize` is small.
In the sample of `selectionSize = 7`, the fixed rewards are [0, 0, 0, 1.5, 5, 75, 1500] which are less than 6553.5.
By considering the possible limit of `selectionSize` is 16, we need bigger values for fixed rewards. As my opinion, we need to give more than 10000 USD to users who match 15 out of 16 numbers.
So, 6553.5 is too small for the range size.