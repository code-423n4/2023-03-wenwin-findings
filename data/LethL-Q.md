# Low Issues

| Number |	Issues Details | Instances |
|---|---|---|
| [L-1] | Using `_safeMint()` instead of `_mint()`	| 1 |
| [L-2] | Owner can renounce Ownership | - |
| [L-3] | Lack of zero address checks | 2 |

## [L-1] Using `_safeMint()` instead of `_mint()`
For the `ERC721` contract, `_mint()` won't check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset.

OpenZeppelin [recommendation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L277) is to use the safe variant of `_mint()`.

Instances(1):
```
File: src/Ticket.sol

26:        _mint(to, ticketId);
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L26

### Recommendation
Replace `_mint()` with `_safeMint()`.

## [L-2] Owner can renounce Ownership
Renouncing ownership results in the contract being left without an owner, removing any functionality that is only available to the owner.

### Recommendation
We recommend either reimplementing the function to disable it or clearly stating whether it is part of the contract design.

## [L-3] Lack of zero address checks
If these variable get configured with address zero, failure to immediately reset the value can result in unexpected behavior for the project.

Instances(2):
```
File: src/staking/StakedTokenLock.sol

18:        stakedToken = IStaking(_stakedToken);
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L18

```
File: src/RNSourceBase.sol

12:        authorizedConsumer = _authorizedConsumer;
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L12

### Reccommendation
Add zero address checks.