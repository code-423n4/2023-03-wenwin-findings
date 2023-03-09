### [G-01] Function should be marked `public` / usage of `this` keyword
claimable() function in https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L159 should be marked `public` instead of `external` as it is called within the contract. Then in https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L261 `this` keyword can be removed from its call. This results in gas savings.

Code:

Lottery.sol#L159:
`function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier)`
Change to:
`function claimable(uint256 ticketId) public view override returns (uint256 claimableAmount, uint8 winTier)`
Lottery.sol#L261:
`(claimedAmount, winTier) = this.claimable(ticketId);`
Change to:
`(claimedAmount, winTier) = claimable(ticketId);`

forge test --gas-report:
before:

`| claimWinningTickets              | 1009            | 15371  | 16054  | 31165  | 6       |`

after:

`| claimWinningTickets              | 1009            | 14646  | 15302  | 29828  | 6       |`