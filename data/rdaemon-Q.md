# Improve readability for other developers
Lottery.sol::receiveRandomNumber
```solidity
uint128 drawFinalized = currentDraw++; // reading the past value and incrementing storage
// One more line makes the code much safer and readable 
uint128 drawFinalized = currentDraw; // reading value for future calculations
currentDraw++; // incrementing storage
```

# Importing files 
Consider importing files explicitly with whatever it is you are using from that file so users know where are constructs declared in the code coming from.
BEFORE:
`import "@openzeppelin/...";`
AFTER:
`import {Contract} from "@openzeppelin/..."`;

# Naming mapping parameters
Since the version of Solidity is 0.8.19, might as well take advantage of naming parameters in mappings.
BEFORE:
`mapping(address => uint256) private frontendDueTicketSales;`
AFTER:
`mapping(address frontendOperator => uint256) private frontendDueTicketSales;`

# Stable Solidity Compiler
Consider changing the Solidity compiler to a more battle-tested version in production. There has been recent discovery of bugs in the compiler.

