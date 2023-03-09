# [01] Use `_safeMint` instead of `mint` for ERC721

[OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d00acef4059807535af0bd0dd0ddf619747a044b/contracts/token/ERC721/ERC721.sol#L277) recommendation is to use `_safeMint()` instead of `mint()` for ERC721.

It is possible that the receiving contract does not support the ERC721 protocol, which can result in the minted NFT to be locked when using `_mint()`.

`_safeMint()` will ensure that the recipient implements IERC721Receiver or is an EOA.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L26

# [02] Emit events before external calls

Multiple functions in the project emit an event as the last statement. Wherever possible, consider emitting events before external calls. In case of reentrancy, funds are not at risk (for external call + event ordering), however emitting events after external calls can damage frontends and monitoring tools in case of reentrancy attacks.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L97-L99

# [03] Missing address zero check on `StakedTokenLock` constructor for `_stakedToken`

If a IStaking gets configured with address zero, failure to immediately reset the value can result in unexpected behavior for the project.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L18

# [04] Reward calculation might not work for tokens that don't use 18 decimals

## Proof of Concept

`Staking.rewardPerToken()` will multiply the unclaimed rewards by 1e18 and divide by the total supply. For a token created with decimals different than 18, the calculation behave unexpectedly. The likelihood will depend of a token with this behavior being used as reward. Similar problem in present on `Staking.earned()`.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L58

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L62

## Impact

Assume USDC is ever used for reward. Then `Staking.rewardPerToken()` will return a bigger value than expected, and rewards will be inflated when calling `Staking.getReward()`. Similarly, if a token with more than 18 decimals is used, the rewards will be smaller than expected.

## Recommended Mitigation Steps

Avoid hardcoding 1e18 in `Staking.rewardPerToken()` and `Staking.earned()`, and call the `.decimals()` function for the token.

# [05] Pragma float

Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L3

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L3

# [06] Lack of checks-effects-interactions

Consider transfering the funds after calling the _mint function in `Staking.stake()`, similarly to how _burn is called before the transfer in `Staking.withdraw()`.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L73-L74

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L85-L86

# [07] Linter item can be solved instead of silenced

The project has demonstrated a good level of maturity and security awareness by checking all linter/static analysis items. However, some items can be solved instead of silenced. This can improve the quality of the codebase overall. 

For example, it's possible to use safeTransfer instead of transfer for the instances highlighted bellow (even if the staked token is trusted - the cost for using safeTransfer is minimal and it will be in sync with the linter - no need for silencing it).

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L32-L34

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L45-L47

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L53-L55  

# [08] Missing unit tests

It is crucial to write tests with possibly 100% coverage for smart contracts.

The following contracts/libraries contain less than 70% branch coverage, after running `forge coverage`.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L12

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L9

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L9

## Recommendation

It is recommended to write tests for all possible code flows.

# [09] Unsafe downcasting

If the value being downcasted is bigger than expected, it can result in a silent overflow and result in accounting errors. Consider using the SafeCast library from OpenZeppelin.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L170

# [10] Replace assert with require or custom error

As described on the solidity [documentation](https://docs.soliditylang.org/en/v0.8.15/control-structures.html#panic-via-assert-and-error-via-require). "The assert function creates an error of type Panic(uint256). … Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix.

Even if the code is expected to be unreachable, consider using a custom error/require statement instead of assert.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99

# [11] Order of functions

The solidity [documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) recommends placing public view functions bellow external mutative functions.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L46-L63

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L65-L124

# [12] First item of array is not included in a loop

If intentional, consider adding a comment near the `winTier` loops when ignoring the first item of the array to illustrate why they're not being processed on the loop. The impact of being non-intentional is leaving the first item and causing issues for processing rewards and random numbers.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L169

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L211

# [13] Add a limit for the maximum number of characters per line

The solidity [documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) recommends a maximum of 120 characters.

Consider adding a limit of 120 characters or less to prevent large lines.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L61

# [14] Avoid repeated validation statements

Repeated statements used for validation can be converted into a function modifier to improve code reusability.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L70

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L82

# [15] Custom error should be capitalized

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L24

# [16] Multiples of ten should use scientific notation.

Using scientific notation for multiples of ten will improve code readability. E.g. 1000 => 1e3

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L8

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L11

# [17] Missing NATSPEC

Consider adding NATSPEC on all external/public functions to improve documentation.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L67

# [18] Named imports can be used

It's possible to name the imports to improve code readability. E.g. `import "@openzeppelin/contracts/token/ERC20/ERC20.sol";` can be rewritten as `import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";`

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L5
