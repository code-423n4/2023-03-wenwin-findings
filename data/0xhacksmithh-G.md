

### [Gas-01] Use of ```uint/int``` smaller than 32bytes incurs overhead 
*instances()*
```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L36
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L37
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L39
``` 
```
File :: src/LotterySetup.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L30-L31
```

### [Gas-02] Multiple address mapping can be combined into a single mapping of an address to a ```struct```, where appropriate.
*instances()*
```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L37
``` 

### [Gas-03] Using ```unchecked``` blocks to save gas.
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.
*instances()*
```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L125
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L172
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L193
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L194
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L205
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L211
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275
``` 
```
File :: src/LotterySetup.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L169
```
```
File :: src/RNSourceController.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L68
```

### [Gas-04] Avoid compound assignment operator in ```state variable```.
Using compound assignment operators for state variable(like state += x, or state -= x) its more expensive than using operator assignment (like state = state + x or, state = state - x) 
*instances()*
```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L129
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L173
``` 

### [Gas-05] Function guaranteed to revert when called by normal user can be marked as ```payable```
*instances()*
```
File :: src/RNSourceController.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L77
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L89

```


### [Gas-06] Avoid contract existance checks by using low level calls.
*instances()*
```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L286
``` 

### [Gas-07] Use require() instead of assert()
*instances()*
```
File :: src/LotterySetup.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147
``` 

### [Gas-08] Setting ```constructor``` to ```payable``` could save gas.
Saves ~13 gas per instances.
*instances()*
```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L84
``` 


### [Gas-09] Use ```public``` function instead of ```modifiers```
*instances()*
```
File :: src/Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L44
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L52
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L60
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L69
``` 