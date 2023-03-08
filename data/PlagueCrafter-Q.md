# Missing NatSpec comments on functions:

**Description:**
As per the solidity documentation that can be found [here](https://docs.soliditylang.org/en/v0.8.19/natspec-format.html): *"It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces" NatSpec includes the formatting for comments that the smart contract author will use, and which are understood by the Solidity compiler. Also detailed below is output of the Solidity compiler, which extracts these comments into a machine-readable format.* (everything in the ABI).". I saw this in `Lottery.sol`, `LotteryMath.sol`, `LotteryToken.sol` among others.   

**Examples:**
- https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L110
- https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L133
- https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L144
- https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L151
- https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L159
- https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L170

**Remediation:**
Add comments to every function across the entire project as per the [NatSpec format](https://docs.soliditylang.org/en/v0.8.19/natspec-format.html)