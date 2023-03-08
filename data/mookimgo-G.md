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

After this change, we can re-use currentRewardPerToken to avoid the `rewardPerToken()` call.