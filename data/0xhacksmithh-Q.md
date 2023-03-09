
### [Low-01] Modifier ```requireValidTicket()``` calling ```isValidTicket()``` on ```uint```
```solidity
    modifier requireValidTicket(uint256 ticket) { 
        if (!ticket.isValidTicket(selectionSize, selectionMax)) { // 
            revert InvalidTicket();
        }
        _;
    }
```
here ticket is uint256 type input parameter on which function call happening.
*instances(1)*
```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L44-L49
``` 

### [Low-02] Solidity ```0.8.19``` is newly released, there may be potential of hidden bugs
Try to use other stable and well tested version of solidity

*instances(all contracts)*
```
File :: 
``` 

### [NC-01] For modern and more readable code, update Import usages
`only import what you need` Specific imports with curly braces allow us to apply this rule better.

**Recommendation:**`import {contract1 , contract2} from "filename.sol";`

```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L7-L11
``` 
```
File :: src/LotterySetup.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L7-L10
```
```
File :: src/RNSourceController.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L6-L7
```

### [NC-02] NatSpec Comments should be increased in contracts
For at least ```public``` and ```external``` functions.

```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L110
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L133
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L144
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L151
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L159
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L234
``` 
```
File :: src/LotterySetup.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L152
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L156
``` 

### [NC-03] OMISSIONS in events
*instances()*
```
File :: src/RNSourceController.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L86
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L102
```

### [NC-04] Using specific notation e.g 1e18 rather than exponentiation e.g 10**18 or 1000000000
*instances(1)*
```
File :: src/LotterySetup.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L81
``` 

### [NC-05] Adding a return statement when the function defines a named return variable is redundant.
*instances()*
```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L239-L246
``` 

### [NC-06] Constant redefine elsewhere
Consider defining ```constants``` in only one contract so that values cannot become out of sync when only one location is updated. A cheap way to store constants in a single location is to create an internal constant in a library.