**src/staking/Staking.sol**
- L54 - The operation lottery.nextTicketId() - lastUpdateTicketId is performed, which will never generate an underflow, therefore it could be wrapped as unchecked and spend less gas.

- L54/55/58 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.

**src/LotterySetup.sol**
- L126/127/128 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.


**src/Lottery.sol**
- L162/163/274/275 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.

- L227/251 - The nextTicketId - lastDrawFinalTicketId operation is performed, which will never generate an underflow, therefore it could be wrapped as unchecked and spend less gas.

- L121/122/124/128/129/130 - Tickets.length is used multiple times in the same function and a variable could be created in memory and avoid constantly consulting the length of tickets.


**src/RNSourceController.sol**
- L93/94/95 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.


**src/LotteryMath.sol**
- L47/48/50/55/128/129 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.


