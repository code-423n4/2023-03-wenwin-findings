## Summary

### Low risk issues
| |Issue|Instances| |
|-|:-|:-:|:-:| 
| [L&#x2011;01] | Use _safeMint instead of _mint | 1 |
| [L&#x2011;02] | Constructor lacks address(0) and (0) value checks | 4 |

### Non-Critical issues
| |Issue|Instances| |
|-|:-|:-:|:-:| 
| [N&#x2011;01] | Use require instead of assert | 2 |
| [N&#x2011;02] | For functions, follow Solidity standard naming conventions | - |
| [N&#x2011;03] | Function writing that does not comply with the Solidity Style Guide | - |
| [N&#x2011;04] | Solidity compiler version should be exactly same in all smart contracts | - |
| [N&#x2011;04] | Do not use floating solidity compiler version. Lock the solidity compiler version | 2 |


### [L&#x2011;01]  Use _safeMint instead of _mint 
OpenZeppelin recommendation is to use the safe variant of _mint.
.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset.
According to openzepplin’s ERC721, the use of _mint is discouraged, use safeMint whenever possible.
[Openzeppelin reference link](https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-_safeMint-address-uint256-bytes-)

```solidity
File: src/Ticket.sol

23    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
24        ticketId = nextTicketId++;
25        ticketsInfo[ticketId] = TicketInfo(drawId, combination, false);
26        _mint(to, ticketId);
27    }
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L23-L27)

### Recommended Mitigation Steps
Use _safeMint whenever possible instead of _mint

### [L&#x2011;02]  Constructor lacks address(0) and (0) value checks
Zero-address check should be used in the constructors, to avoid the risk of setting a storage variable as address(0) at deploying time. In RNSourceBase contract, authorizedConsumer is immutable therefore any address error in constructor will result in redeployment of contract.

There are 4 instances of this issue.

```solidity
File: src/RNSourceBase.sol

11    constructor(address _authorizedConsumer) {
12        authorizedConsumer = _authorizedConsumer;
13    }
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceBase.sol#L11-L13)

```solidity
File: src/staking/StakedTokenLock.sol

16    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
17        _transferOwnership(msg.sender);
18        stakedToken = IStaking(_stakedToken);
19        rewardsToken = stakedToken.rewardsToken();
20        depositDeadline = _depositDeadline;
21        lockDuration = _lockDuration;
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L16-L21)

### [N&#x2011;01]  Use require instead of assert 
### Description
Assert should not be used except for tests, require should be used.
Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

### assert() and require()
The assert() and require() functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.
They are quite similar as both check for conditions and if they are not met, would throw an error.
The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made. It does not refund gas. 
Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256).Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. There’s a mistake”.
There are 2 instances of this issue.

```solidity
File: src/LotterySetup.sol

147        assert(initialPot > 0);
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L147)

```solidity
File: src/TicketUtils.sol

99            assert((winTier <= selectionSize) && (intersection == uint256(0)));
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L99)

### [N&#x2011;02]  For functions, follow Solidity standard naming conventions
Proper use of _ as a function name prefix should be taken care and a common pattern is to prefix internal and private function names with _.
This _ pattern is applied in LotterySetup.sol, Staking.sol only however, It is recommended to apply this pattern in other smart contracts.

### [N&#x2011;03]  Function writing that does not comply with the Solidity Style Guide
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

[Link to Solidity style-guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html)

Functions should be grouped according to their visibility and ordered:
constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

### [N&#x2011;03]  Solidity compiler version should be exactly same in all smart contracts
Solidity compiler version should be exactly same in all smart contracts. Most of the smart contracts have used the latest solidity version 0.8.19 whereas StakedTokenLock.sol uses ^0.8.17 and VRFv2RNSource.sol uses ^0.8.7.

```solidity
File: src/staking/StakedTokenLock.sol

3     pragma solidity ^0.8.17;
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L3)

```solidity
File: src/VRFv2RNSource.sol

3     pragma solidity ^0.8.7;
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/VRFv2RNSource.sol#L3)

### [N&#x2011;04]  Do not use floating solidity compiler version. Lock the solidity compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
There are 2 instances of this issue.

```solidity
File: src/staking/StakedTokenLock.sol

3     pragma solidity ^0.8.17;
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L3)

```solidity
File: src/VRFv2RNSource.sol

3     pragma solidity ^0.8.7;
```
[Link to code](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/VRFv2RNSource.sol#L3)
