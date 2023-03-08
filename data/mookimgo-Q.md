#1. Ticket should add an tokenURI for displaying ticketsInfo

Ticket.sol does not has custom tokenURI function, which leads to incorrect NFT image display.

Suggestion: Add a tokenURI function to render `drawId`, `combination`, and the most important `claimed`. This can be done in solidity by concatenating svg lines and use base64 encode.

