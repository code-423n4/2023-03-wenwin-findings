# Findings Summary

| ID     | Title                                                                               | Severity          |
| ------ | ----------------------------------------------------------------------------------- | ----------------- |
| [G-01] | Setting the constructor to payable                                                  | Gas Optimizations |
| [G-02] | Functions guaranteed to revert when callled by normal users can be marked `payable` | Gas Optimizations |
| [G-03] | <x> += <y> costs more gas than <x> = <x> + <y> for state variables (-= too)         | Gas Optimizations |
| [G-04] | Use nested if and, avoid multiple check combinations                                | Gas Optimizations |

# Detailed Findings

# [G-01] Setting the constructor to `payable`



## Description

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

## context

```solidity
file: src/LotteryToken.sol :

constructor() ERC20("Wenwin Lottery", "LOT") {
        //is it ok to not check if for address zero ?
        owner = msg.sender;
        _mint(msg.sender, INITIAL_SUPPLY);
    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryToken.sol#L17

```solidity
file: src/VRFv2RNSource.sol

constructor(
        address _authorizedConsumer,
        address _linkAddress,
        address _wrapperAddress,
        uint16 _requestConfirmations,
        uint32 _callbackGasLimit
    )
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/VRFv2RNSource.sol#L13

```solidity
file: src/staking/StakedTokenLock.sol

constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration)
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L16

```solidity
file: src/staking/Staking.sol

 constructor(
        ILottery _lottery,
        IERC20 _rewardsToken,
        IERC20 _stakingToken,
        string memory name,
        string memory symbol
    )
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L16

```solidity
file: src/Lottery.sol

constructor(
LotterySetupParams memory lotterySetupParams,
uint256 playerRewardFirstDraw,
uint256 playerRewardDecreasePerDraw,
uint256[] memory rewardsToReferrersPerDraw,
uint256 maxRNFailedAttempts,
uint256 maxRNRequestDelay
)
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L84

```solidity
file: src/LotterySetup.sol
 constructor(LotterySetupParams memory lotterySetupParams)
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L41

```solidity
file: src/RNSourceBase.sol

constructor(address _authorizedConsumer) {
        authorizedConsumer = _authorizedConsumer;
    }
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceBase.sol#L11

```solidity
file: src/RNSourceController.sol

 constructor(uint256 _maxFailedAttempts, uint256 _maxRequestDelay)

```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L26

## Recommendations

add `payable` to the constructors

# [G-02] Functions guaranteed to revert when callled by normal users can be marked `payable`

## Description

If a function `modifier` or require such as onlyOwner-admin is used,
the function will revert if a normal user tries to pay the function.
Functions guaranteed to revert when called by normal users can be
marked payable (for onlyOwner, onlyTicketOwner).

## context

```solidity
file: src/Staking/StakedTokenLock.sol

 function deposit(uint256 amount) external override onlyOwner
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L24

```solidity
file: src/Staking/StakedTokenLock.sol

  function withdraw(uint256 amount) external override onlyOwner
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L37

```solidity
file: src/Lottery.sol
function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount)
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L259

## Recommendations

add `payable` to these functions

# [G-03] <x> += <y> costs more gas than <x> = <x> + <y> for state variables (-= too)

## Description

using the addition operator instead of plus-equals saves gas. Subtractions act the same way

## context

```solidity
file: src/Staking/StakedTokenLock.sol

 depositedBalance += amount;
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L30

```solidity
  file: src/Lottery.sol

   claimedAmount += claimWinningTicket(ticketIds[i])
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L173

```solidity
  file: src/Lottery.sol

 currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L275

```solidity
file: src/LotteryMath.sol

newProfit -= int256(expectedRewardsOut);
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L55

```solidity
file: src/LotteryMath.sol

excessPotInt -= int256(fixedJackpotSize);
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L64

```solidity
file: src/LotteryMath.sol

 bonusMulti += (excessPot * EXCESS_BONUS_ALLOCATION) / (ticketsSold * expectedPayout);
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L84

## Recommendations

use x = x + y instead of x+=y

# [G-04] Use nested if and, avoid multiple check combinations

## Description

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

## context

```solidity
  file: src/LotterySetup.sol

    if (
            lotterySetupParams.expectedPayout < lotterySetupParams.ticketPrice / 100
                || lotterySetupParams.expectedPayout >= lotterySetupParams.ticketPrice
        ) {
            revert InvalidExpectedPayout();
        }
        if (
            lotterySetupParams.selectionSize > 16 || lotterySetupParams.selectionSize >= lotterySetupParams.selectionMax
        ) {
            revert SelectionSizeTooBig();
        }
        if (
            lotterySetupParams.drawSchedule.drawCoolDownPeriod >= lotterySetupParams.drawSchedule.drawPeriod
                || lotterySetupParams.drawSchedule.firstDrawScheduledAt < lotterySetupParams.drawSchedule.drawPeriod
        )
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L54
