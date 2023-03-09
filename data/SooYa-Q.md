# Should add check `amount` is greater than `depositedBalance`
The `withdraw` function could be failed to call If the `depositedBalance` is less than the `amount`, due to underflow. I recommend to check the 'depositedBalance' first before the subtraction.
Below is the following instance for this issue:

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L37-L48

Add the line code below before the subtraction code:
```
if (depositedBalance < amount) {
	revert("Insufficient Balance");
	}
```

# Use `require()` instead `assert()`
The Solidity assert() function is meant to assert invariants. Properly functioning code should never reach a failing assert statement.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99
```
assert((winTier <= selectionSize) && (intersection == uint256(0)));
```

Recommended: Consider whether the condition checked in the assert() is actually an invariant. If not, replace the assert() statement with a require() statement.


