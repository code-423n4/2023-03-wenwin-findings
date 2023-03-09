## Non-Critical Issues List
Issue 
### [N-01] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
### [N-02] PRAGMA VERSION ^0.8.17 and 0.8.19 VERSION TOO RECENT TO BE TRUSTED.
### [N-03] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES
### [N-04] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS
### [N-05] MISSING EVENT FOR CRITICAL PARAMETERS INIT AND CHANGE
### [N-06] USE REQUIRE INSTEAD OF ASSERT
### [N-07] It's better to emit after all processing is done
Total:  07 issues

## [N-01] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values):

This will help with readability and easier maintenance for future changes.
constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution

## [N-02] PRAGMA VERSION ^0.8.17 and 0.8.19 VERSION TOO RECENT TO BE TRUSTED.
https://github.com/ethereum/solidity/blob/develop/Changelog.md

## [N-03] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES
Description:
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
Recommendation:
import {contract1 , contract2} from "filename.sol";
A good example from the ArtGobblers project;
```
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```

## [N-04] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
Recommendation:
NatSpec comments should be increased in contracts

## [N-05] MISSING EVENT FOR CRITICAL PARAMETERS INIT AND CHANGE
Description:
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Recommendation:
Add Event-Emit
File: src/staking/StakedTokenLock.sol
```
24:     function deposit(uint256 amount) external override onlyOwner {
37:    function withdraw(uint256 amount) external override onlyOwner {
50:    function getReward() external override {
```

## [N-06] USE REQUIRE INSTEAD OF ASSERT
Assert should not be used except for tests, require should be used
Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.
assert() and reqire();
The big difference between the two is that the assert()function when false, uses up all the remaining gas and reverts all the changes made.
Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay.This is the most common Solidity function used by developers for debugging and error handling.
Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256). Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there’s a mistake”.
File: src/LotterySetup.sol
```
147:        assert(initialPot > 0);
```

## [N-07] It's better to emit after all processing is done
File: src/Lottery.sol 
```
155:        emit ClaimedRewards(beneficiary, claimedAmount, rewardType);
```

## Low Risk Issues List
Issue 
### [L-01] MISSING REENTRANCY GUARD TO WITHDRAW FUNCTION
### [L-02]  _safeMint() should be used rather than _mint() wherever possible
Total:  02 issues

## [L-01] MISSING REENTRANCY GUARD TO WITHDRAW FUNCTION
https://www.paradigm.xyz/2021/08/the-dangers-of-surprising-code
Use Openzeppelin or Solmate Re-Entrancy pattern.
Here is a example of a re-entrancy guard
File: src/staking/Staking.sol
```
79:    function withdraw(uint256 amount) public override {
```

## [L-02]  _safeMint() should be used rather than _mint() wherever possible
_mint() is discouraged (https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin (https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) and solmate (https://github.com/transmissions11/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180) have versions of this function
File: src/staking/Staking.sol
```
74:         _mint(msg.sender, amount);
```

## Suggestions List
### [S-01] GENERATE PERFECT CODE HEADERS EVERY TIME
Total: 01 issues

## [S-01] GENERATE PERFECT CODE HEADERS EVERY TIME
Description:
I recommend using header for Solidity code layout and readability
https://github.com/transmissions11/headers
```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```
