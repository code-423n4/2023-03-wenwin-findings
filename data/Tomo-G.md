# Gas Report

## ✅ G-1: Packing of storage variables
### 📝 Description

Pack variables in one slot by defining them in a lower data type. Packing only helps when multiple variables of the packed slot are being accessed in the same call. If not done correctly, it increases gas costs instead, due to shifts required.

### 💡 Recommendation
- before **227971** gas (Enable optimization: 300)

```solidity=
// SPDX-License-Identifier: UNLICENSED
// slither-disable-next-line solc-version
pragma solidity 0.8.19;

contract Lottery /*... */{
    /*... */

    uint256 private claimedStakingRewardAtTicketId;
    mapping(address => uint256) private frontendDueTicketSales;
    mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;

    address public immutable stakingRewardRecipient;

    uint256 public lastDrawFinalTicketId;

    bool public drawExecutionInProgress;
    uint128 public currentDraw;

    mapping(uint128 => uint120) public winningTicket;
    mapping(uint128 => mapping(uint8 => uint256)) public winAmount;

    mapping(uint128 => uint256) public ticketsSold;
    int256 public currentNetProfit;

    // @dev: this is a sample constructor
    constructor(address _stakingRewardRecipient) public {
        stakingRewardRecipient = _stakingRewardRecipient;
    }
}
```

- after **226011** gas(Enable optimization: 300) 

"This means you can save **1960** gas."

``` solidity
contract Lottery /*... */{
    /*... */
    
    /* Mapping */
    mapping(address => uint256) private frontendDueTicketSales;
    mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
    mapping(uint128 => uint120) public winningTicket;
    mapping(uint128 => mapping(uint8 => uint256)) public winAmount;
    mapping(uint128 => uint256) public ticketsSold;

    /* Variables */
    uint256 private claimedStakingRewardAtTicketId; //slot0
    address public immutable stakingRewardRecipient; // slot1
    uint256 public lastDrawFinalTicketId; //slot2
    uint128 public drawExecutionInProgress; //slot3 // changed
    uint128 public currentDraw; // slot3
    int256 public currentNetProfit; //slot4

    // @dev: this is a sample constructor
    constructor(address _stakingRewardRecipient) public {
        stakingRewardRecipient = _stakingRewardRecipient;
    }
}
```
### 🔍 Findings:
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L21-L40
