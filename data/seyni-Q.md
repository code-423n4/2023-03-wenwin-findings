# [L-01] `randomNumber` is divided by 35 instead of 256 (shr 8) when retrieving numbers from chainlink random number

## Vulnerability details
The following comment suggests that at each iteration when determining the seven selected numbers from the chainlink random number we "shift it for 8 bits to the right".
[TicketUtils.sol#L37](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L37)
```
    /// In each iteration, we calculate the modulo of a random number and then shift it for 8 bits to the right.
```

However, what is actually done is that the chainlink random number is divided by `currentSelectionCount == 35`, as you can see in the following code:
[TicketUtils.sol#L59](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L59)
```
            randomNumber /= currentSelectionCount;
```

## Impact
Every numbers except the first one are going to be different from what they would have been using shr 8. Using shr 8 preserve the bits of the randomness created by chainlink vrf, whereas dividing by 35,34,33 etc, doesn't.

## References
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L43

## Recommended Mitigation Steps
Consider shifting the bits right 8 times instead of dividing by `currentSelectionCount`.

# [L-02] `_mint` is used instead of `_safeMint` for ERC721

## Vulnerability details

`_mint` is used instead of `_safeMint` to mint NFT tickets. Using `_mint` prevent reentrancy attack, but leave the risk that a contract calling a function (here `buyTickets`) which call `_mint` will be sent an NFT regardless of whether or not it signaled that he can receive them by using the `ERC721TokenReceiver` interface.

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
Any NFT minted by a contract which doesn't implement the interface `ERC721TokenReceiver` and call `buyTickets` could end up frozen is the calling contract. This could only happen if someone implements a contract that call the Lottery contract without taking care of implementing a function to send NFT out of the contract. Therefore, it can only happen if the person implementing the contract doesn't know that it will get minted an NFT.


## References
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L26

## Recommended Mitigation Steps
Consider using `_safeMint` instead of `_mint` and use a reentrancy guard for any function calling `Ticket.mint`.