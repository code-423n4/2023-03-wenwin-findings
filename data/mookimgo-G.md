# 1. requestId is unique by design, no need to check

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L22-L25

```
        uint256 requestId = requestRandomnessFromUnderlyingSource();
        if (requests[requestId].status != RequestStatus.None) {
            revert requestIdAlreadyExists(requestId);
        }
```

As the requestId given by ChainLink is a hash including incrementing nonce:

https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/VRFCoordinatorV2.sol#L410-L411

```
requestId is a hash of (keyHash, msg.sender, subId, nonce);
in which nonce is incremented for every request
```

So, it's guaranteed that requestId is unique, we can remove this `requests[requestId].status != RequestStatus.None` check to save gas

# 2. rewards[account] update can save gas by not calling `earned`

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L122

```
rewards[account] = earned(account);
```

Each `stake`, `withdraw`, `getReward` call will call this `_updateReward` function, which will use `earned(account)` to update `rewards[account]`.

But the `earned` function will call `rewardPerToken()` again, which is an unnecessary high gas cost operation because it contains at least two external calls.

Suggestion: change to:

```
    function _updateReward(address account) internal {
        uint256 currentRewardPerToken = rewardPerToken();
        rewardPerTokenStored = currentRewardPerToken;
        lastUpdateTicketId = lottery.nextTicketId();
        rewards[account] = balanceOf(account) * (currentRewardPerToken - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];
        userRewardPerTokenPaid[account] = currentRewardPerToken;
    }
```

After this change, we can re-use currentRewardPerToken to avoid the unnecessary `rewardPerToken()` call.

# 3. rewardToken's decimals should be cached as an immutable variable in LotterySetup's constructor

ERC20's decimals should be a constant, so there's no need to call `rewardToken.decimals()` each time. Besides, `rewardToken` is an immutable variable, so adding an immutable variable for rewardToken's decimals makes sense.

`rewardToken.decimals()` is called in:

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L128
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L168

And both code use `10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1)`, so we can cache it to save gas for the external call, the minus, and the exp operation gas.

Suggestion: 

- add `uint256 private immutable override rewardTokenFactor;`
- add `rewardTokenFactor = 10 ** (IERC20Metadata(address(rewardToken)).decimals()-1);` to constructor
- change line 128 to `return extracted * rewardTokenFactor;`
- change line 168 to `uint256 divisor = rewardTokenFactor;`