# Gas Findings

## [G-01] `claimable` should be marked public

- https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Lottery.sol#L261

`claimWinningTicket` calls `this.claimable(ticketId);` because `claimable` is marked `external`. If we mark it as public `claimWinningTicket` will cost less gas.

## [G-02] `withdraw` in `StakedTokenLock.sol` doesn’t need to check `block.timestamp > depositDeadline`

- https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/StakedTokenLock.sol#L39

It’s only possible to deposit if `block.timestamp < depositDeadline` (see the deposit function). So we can assume that the timestamp for the withdraw is after the depositDeadline and we don’t need to check it.

## [G-03] You don’t need to read `unclaimedTickets` from storage in `claimPerDraw`

- https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/ReferralSystem.sol#L145

In this line we see the following code:

```solidity
_unclaimedTickets = unclaimedTickets[drawId][msg.sender];
```

This is unnecessary as we read this from storage a few lines up. The only difference that could have happened was that referrerTicketCount was modified, but we never access that again.