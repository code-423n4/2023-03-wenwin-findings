# There are multiple instances where arithmetic operations can be unchecked to save gas. 

[Lottery.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L181). In the functions `receiveRandomNumber()` `claimWinningTickets()` and `buyTickets()` `++i;` can be unchecked to save gas. For example in the function `buyTickets()`.


```
        for (uint256 i = 0; i < drawIds.length) {
           
            ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
        }

```
Optimized to be.

```
 for (uint256 i = 0; i < drawIds.length) {
           
            ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
            unchecked{++i};

        }
```

Below are the instances were it is occuring.
```
ReferralSystem.sol
 constructor()
 function claimReferralReward()
Lottery.sol
 function receiveRandomNumber()
 function buyTickets()
 function claimWinningTickets()
TicketUtils.sol
 function isValidTicket()
 function reconstructTicket()
 function ticketWinTier()
```



## Using !=0 is more gas efficient than > 0 for unsigned integers.
Consider using !=0 when checking if a value isn't 0 for unsigned intergers. Below are the instances where it is occuring.
```
LotterySetup.sol
 function finalizeInitialPotRaise()
ReferralSystem.sol
 function referralDrawFinalize()
 function claimPerDraw()
Staking.sol
 function getReward() 
```
# Immutable variables should be used for expressions, constants should be used for literal values
While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.
Constants should be used only for literals while immutables should be used for expressions or arguments passed to the constructor.

```
LotteryMath.sol
    uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;
    uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;
    uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;
    uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;
    uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;
```

# An array’s length should be cached to save gas in for-loops
Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

Caching the array length in the stack saves around 3 gas per iteration.

Below are instances where it occurs. Its recommended to store the array's length to a variable before the for-loop and use that instead.
```
LotteryMath.sol
 function buyTickets()
ReferralSystem.sol
 function claimReferralReward()

```

