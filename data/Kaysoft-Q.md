## [NC-1] USE NAMED IMPORT 
Consider using named import like `import {ERC20} from "./ERC20.sol"` instead of global `import "./ERC20.sol"`

This ensures The specific required module from the file is imported and not every module from the file.

Files: all files


## [NC-2] ADD NATSPEC COMMENTS TO FUNCTIONS AND CONSTRUCTORS
see: https://docs.soliditylang.org/en/v0.8.17/natspec-format.html
Files: Most of the functions in the files.


## [NC-3] REMOVE UNNAMED PARAMETER OF THE `beforeTokenTransfer` function
The third argument in this function is not named and unused. You may want to add comments why it is so.
File: 
- [src/staking/Staking.sol#L108](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L108)
```Solidity
function _beforeTokenTransfer(address from, address to, uint256) internal override {
        if (from != address(0)) {
            _updateReward(from);
        }

        if (to != address(0)) {
            _updateReward(to);
        }
    }

```

## [L-01] Use `_safeMint` instead of `_mint`

The use of `_safeMint` is encouraged which ensures the receiver address is either an EOA or a contract that implements the IERC721Receiver.

File: 
- [src/Ticket.sol#L26](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L26)

```Soldity
src/Ticket.sol#L26

function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
        ticketId = nextTicketId++;
        ticketsInfo[ticketId] = TicketInfo(drawId, combination, false);
        _mint(to, ticketId);
    }

```
