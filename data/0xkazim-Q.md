# Findings Summary

| ID     | Title                                                   | Severity |
| ------ | ------------------------------------------------------- | -------- |
| [L-01] | Setting the constructor to payable                      | Low      |
| [L-02] | A single point of failure                               | Low      |
| [L-03] | Use the safe variant and ERC721.mint                    | Low      |
| [L-04] | `getReward` can be called by everyone and front running | Low      |

# Detailed Findings

# [L-01] [ Lack of zero address checks for `RNSourceBase.sol` constructor and `stakedToken.sol` constructor !

## Description

If these variable get configured with address zero, failure to immediately reset the value can result in unexpected behavior for the project.

## context

```solidity
file: src/RNSourceBase.sol :

constructor(address _authorizedConsumer) {
        authorizedConsumer = _authorizedConsumer;
    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceBase.sol#L11

```solidity
file: src/staking/stakedToken.sol

constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
        _transferOwnership(msg.sender);
        stakedToken = IStaking(_stakedToken);
        rewardsToken = stakedToken.rewardsToken();
        depositDeadline = _depositDeadline;
        lockDuration = _lockDuration;
    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L16

# I think it's better to doing zero address check for `buyTickets` if the address of frontend and referrer is set manualy

```solidity
file: src/Lottery.sol

function buyTickets(
        uint128[] calldata drawIds,
        uint120[] calldata tickets,
        address frontend,
        address referrer
    )
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L110

## Recommendations

add zero check address to the code above !

# [L-02] A single point of failure

## Description

The onlyOwner role has a single point of failure and onlyOwner can use critical a few functions. Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project

## context

```solidity
file: src/Staking/stakedToken.sol :

function deposit(uint256 amount) external override onlyOwner {
        // slither-disable-next-line timestamp
        if (block.timestamp > depositDeadline) {
            revert DepositPeriodOver();
        }

        depositedBalance += amount;

        // No need for SafeTransferFrom, only trusted staked token is used.
        // slither-disable-next-line unchecked-transfer
        stakedToken.transferFrom(msg.sender, address(this), amount);
    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L24

```solidity
file: src/staking/stakedToken.sol

function withdraw(uint256 amount) external override onlyOwner {
        // slither-disable-next-line timestamp
        if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {
            revert LockPeriodOngoing();
        }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L37

## Recommendations

Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

# [L-03] Use the safe variant and ERC721.mint

## Description

`.mint` won’t check if the recipient is able to receive the NFT. If an incorrect address is passed,
it will result in a silent failure and loss of asset.

## context

```solidity
file: src/Ticket.sol

function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
        ticketId = nextTicketId++;
        ticketsInfo[ticketId] = TicketInfo(drawId, combination, false);
        _mint(to, ticketId);
    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L23

## Recommendations

use `_safemint` instead of `_mint` of openzipplin

# [L-04] `getReward` can be called by everyone and front running

## Description

maybe this function can be front runnig by someone because everyone can call it and cause a frontend running!

## context

```solidity
file: src/staking/StakedTokenLock.sol

function getReward() external override {
        stakedToken.getReward();

        // No need for SafeTransfer, only trusted reward token is used.
        // slither-disable-next-line unchecked-transfer
        rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));
    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L50

## Recommendations

use commit-reveal scheme (https://medium.com/swlh/exploring-commit-reveal-schemes-on-ethereum-c4ff5a777db8)
use submarine send (https://libsubmarine.org/)
