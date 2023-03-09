## 1. STATE VARIABLES SHOULD BE CACHED IN STACK VARIABLES RATHER THAN RE-READING THEM FROM STORAGE 

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.
SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

In the `receiveRandomNumber()` function of the `Lottery.sol` the `selectionSize` state variable is read multiple times from storage.
Instead it can be read only once from storage and then cached it to a memory variable after which the value can be read from the memory which is gas efficient.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L203-L221
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L272-L273

## 2. MEMORY VARIABLE IS DECLARED BUT NOT USED, HENCE CAN BE REMOVED

The memory variable `winTier` is declared as a memory variable inside the `claimWinningTicket()` function in `lottery.sol` contract.
But the `winTier` variable is never used inside the function scope and hence can be removed to save gas.

        uint256 winTier;
        (claimedAmount, winTier) = this.claimable(ticketId); 
		
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L259-L261

## 3. USE ASSEMBLY TO CHECK FOR `address(0)` THIS WILL SAVE 6 GAS PER INSTANCE

        if (address(_lottery) == address(0)) {
            revert ZeroAddressInput();
        }
		
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L30-L39