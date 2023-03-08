## Add unchecked {} for addition where the operands cannot overflow
```solidity
src/Ticket.sol:

24:     ticketId = nextTicketId++;
```
## Use assembly to check for address(0)
```solidity
src/staking/Staking.sol:

31:     if (address(_lottery) == address(0))
34:     if (address(_lottery) == address(0))
37:     if (address(_lottery) == address(0))

src/RNSourceController.sol:

78:        if (address(rnSource) == address(0))
81:        if (address(source) != address(0))


```

## A double call to `_updateRewards` in `exit` method

The `exit` method in [Staking](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol) first calls
```solidity
src/staking/Staking.sol:

104:    withdraw(balanceOf(msg.sender));
```
where `_updateRewards` called during _burn call
```solidity
src/staking/Staking.sol:

85:     _burn(msg.sender, amount);
```
after in `exit` call `getReward` method which also call `_updateRewards` for `msg.sender`
```solidity
src/staking/Staking.sol:

92:     _updateReward(msg.sender);
```
### Recommendation
* Change the implementation of the `exit` method, which will not call `_updateReward` 2 times.

 Example:
create inner `_getReward()` which implement standard flow and 

`getReward` will next:
```solidity
    function getReward() external override {
        _updateReward(msg.sender);
        _getReward()
    }
```
`exit` will next:
```solidity
    function exit() external override {
        withdraw(balanceOf(msg.sender));
        _getReward();
    }
```
## Cached `lottery.nextTicketId()` in `_updateReward` method for use in `lastUpdateTicketId()` and `earned()` inner methods 
Many calls `lottery.nextTicketId` are made within a single function `_updateReward`

During `_updateRewards` we can see the next places where `lottery.nextTicketId()` is called
```solidity 
src/staking/Staking.sol:

118:    function _updateReward(address account) internal {
119:            uint256 currentRewardPerToken = rewardPerToken(); // *1
120:            rewardPerTokenStored = currentRewardPerToken;
121:            lastUpdateTicketId = lottery.nextTicketId(); // *2
122:            rewards[account] = earned(account); // *3
123:            userRewardPerTokenPaid[account] = currentRewardPerToken;
124:        }
```

`rewardPerToken` ->
```solidity
src/staking/Staking.sol:

54:     uint256 ticketsSoldSinceUpdate = lottery.nextTicketId() - lastUpdateTicketId;
```

`earned` -> call `rewardPerToken` again
```solidity
src/staking/Staking.sol:

62:        return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];
```

### Recommendation
* Cache `lottery.nextTicketId()` and pass it to internal function calls as a parameter

## Optimization of calculation flow when any ticket not sold
In the event that tickets were not sold, calculations and actions on overwriting are still performed
```solidity
src/staking/Staking.sol:

118:    function _updateReward(address account) internal {
119:        uint256 currentRewardPerToken = rewardPerToken(); //*
120:        rewardPerTokenStored = currentRewardPerToken; //*
121:        lastUpdateTicketId = lottery.nextTicketId(); //*
122:        rewards[account] = earned(account);
123:        userRewardPerTokenPaid[account] = currentRewardPerToken;
124:    }
```
### Recommendation
* if tickets have not been sold `lottery.nextTicketId == lastUpdateTicketId` since the last update, then skip some the calculations stages
  example:
    - not need update `lastUpdateTicketId`
    - not need calculate new `rewardPerToken`
    - ...
