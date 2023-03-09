# Qa

## [L-1 ]Unsafe Transfer

ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard. To account for this, either use OpenZeppelin's SafeERC20 library or wrap each operation in a require statement.
        Additionally, ERC20's approve functions have a known race-condition vulnerability. To account for this, use OpenZeppelin's SafeERC20 library's `safeIncrease` or `safeDecrease` Allowance functions.

### Instances

- StakedTokenLock.sol:34

```solidity
stakedToken.transferFrom(msg.sender, address(this), amount);
```

[https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L34](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L34)

- StakedTokenLock.sol:47

```solidity
stakedToken.transfer(msg.sender, amount);
```

[https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L47](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L47)

- StakedTokenLock.sol:55

```solidity
rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));
```

[https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L55](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L55)

## [N-1] Costly Operations in if loop

```solidity
function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
        ticketId = nextTicketId++;
        ticketsInfo[ticketId] = TicketInfo(drawId, combination, false);
        _mint(to, ticketId);
```

[https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L23-L26](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L23-L26)