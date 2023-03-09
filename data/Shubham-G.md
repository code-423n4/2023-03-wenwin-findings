[G-01] - Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

File:		src/staking/StakedTokenLock.sol
39:             if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) 

File:           src/LotteryMath.sol               
83:             if (excessPot > 0 && ticketsSold > 0)

Instead, use the following. They show a decrease in gas consumed in gas-coverage report.

File:		src/staking/StakedTokenLock.sol
39:             if (block.timestamp > depositDeadline) {
                  if (block.timestamp < depositDeadline + lockDuration) {

File:           src/LotteryMath.sol 
83:              if (excessPot > 0) {
                   if (ticketsSold > 0) {