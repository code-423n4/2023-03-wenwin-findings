# [L-02] `_mint` is used instead of `_safeMint` for ERC721

## Vulnerability details

`_mint` is used instead of `_safeMint` to mint NFT tickets. Using `_mint` prevent reentrancy attack, but leave the risk that a contract calling a function (here `buyTickets`) which call `_mint` will be sent an NFT regardless of whether or not it signaled that he can received them by using the `ERC721TokenReceiver` interface.

According to the documentation of ERC721.sol by Openzeppelin:
[ERC721.sol#L274-L286](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/3d7a93876a2e5e1d7fe29b5a0e96e222afdc4cfa/contracts/token/ERC721/ERC721.sol#L274-L286)
```solidity
/**
 * @dev Mints `tokenId` and transfers it to `to`.
 *
 * WARNING: Usage of this method is discouraged, use {_safeMint} whenever possible
 *
 * Requirements:
 *
 * - `tokenId` must not exist.
 * - `to` cannot be the zero address.
 *
 * Emits a {Transfer} event.
 */
function _mint(address to, uint256 tokenId) internal virtual {
```

## Impact
Any NFT minted by a contract which doesn't implement the interface `ERC721TokenReceiver` and call `buyTickets` could end up frozen is the calling contract. This could only happen if someone implement a contract that call the Lottery contract without taking care of implementing a function to send NFT out of the contract. Therefore it can only happen if the person implementing the contract doesn't know that it will get minted a NFT.


## References
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L26

## Recommended Mitigation Steps
Consider using `_safeMint` instead of `_mint` and use a reentrancy guard for any function calling `Ticket.mint`.