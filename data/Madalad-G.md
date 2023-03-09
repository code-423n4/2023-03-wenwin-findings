# Gas Optimizations Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[G-01]|Functions guaranteed to revert when called by normal users can be marked `payable`|4|
|[G-02]|Use assembly to check for `address(0)`|10|
|[G-03]|Cache storage variables rather than re-reading from storage|5|
|[G-04]|Inline `internal` functions that are only called once|7|
|[G-05]|Use `unchecked` for operations that cannot overflow/underflow|11|
|[G-06]|Pack state variables into fewer storage slots|1|
|[G-07]|Use `private` rather than `public` for constants|9|
|[G-08]|Change `public` functions to `external`|2|
|[G-09]|Use a more recent version of Solidity|2|
|[G-10]|Naming a return variable and still calling the `return` keyword wastes gas|11|
|[G-11]|`x += y` costs more gas than `x = x + y` for state variables|3|
|[G-12]|Usage of `uint` smaller than 32 bytes (256 bits) incurs overhead|17|
|[G-13]|`++i` costs less gas than `i++`|7|

&nbsp;
Total issues: 13
Total instances: 89
&nbsp;
# Gas Optimizations
## [G-01] Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost (2400 per instance).

Instances: 4
- [src/RNSourceController.sol#L79](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L79)
- [src/RNSourceController.sol#L91](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L91)
- [src/staking/StakedTokenLock.sol#L24](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L24)
- [src/staking/StakedTokenLock.sol#L37](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L37)

&nbsp;
## [G-02] Use assembly to check for `address(0)`

Saves 16000 deployment gas per instance and 6 runtime gas per instance.

```solidity
assembly {
 if iszero(_addr) {
  mstore(0x00, "zero address")
  revert(0x00, 0x20)
 }
}
```

Instances: 10
- [src/LotterySetup.sol#L42](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L42)
- [src/RNSourceController.sol#L80](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L80)
- [src/RNSourceController.sol#L83](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L83)
- [src/RNSourceController.sol#L92](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L92)
- [src/ReferralSystem.sol#L60](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L60)
- [src/staking/Staking.sol#L31](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L31)
- [src/staking/Staking.sol#L34](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L34)
- [src/staking/Staking.sol#L37](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L37)
- [src/staking/Staking.sol#L109](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L109)
- [src/staking/Staking.sol#L113](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L113)

&nbsp;
## [G-03] Cache storage variables rather than re-reading from storage

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

Instances: 5
- [src/RNSourceController.sol#L115](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L115)
- [src/RNSourceController.sol#L117](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L117)
- [src/RNSourceController.sol#L119](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L119)
- [src/Lottery.sol#L136](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L136)
- [src/Lottery.sol#L142](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L142)

&nbsp;
## [G-04] Inline `internal` functions that are only called once

Saves 20-40 gas per instance. See https://blog.soliditylang.org/2021/03/02/saving-gas-with-simple-inliner/ for more details.

Instances: 7
- [src/LotterySetup.sol#L122](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L122)
- [src/RNSourceBase.sol#L22](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L22)
- [src/RNSourceController.sol#L55](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L55)
- [src/ReferralSystem.sol#L81](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L81)
- [src/ReferralSystem.sol#L137](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L137)
- [src/ReferralSystem.sol#L103](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L103)
- [src/ReferralSystem.sol#L96](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L96)

&nbsp;
## [G-05] Use `unchecked` for operations that cannot overflow/underflow

By bypassing Solidity's built in overflow/underflow checks using `unchecked`, we can save gas. This is especially beneficial for the index variable within for loops (saves 120 gas per iteration).

Instances: 11
- [src/LotterySetup.sol#L170](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L170)
- [src/Lottery.sol#L126](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L126)
- [src/Lottery.sol#L173](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L173)
- [src/Lottery.sol#L212](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L212)
- [src/ReferralSystem.sol#L35](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L35)
- [src/ReferralSystem.sol#L77](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L77)
- [src/TicketUtils.sol#L28](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L28)
- [src/TicketUtils.sol#L57](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L57)
- [src/TicketUtils.sol#L65](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L65)
- [src/TicketUtils.sol#L68](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L68)
- [src/TicketUtils.sol#L95](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L95)

&nbsp;
## [G-06] Pack state variables into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.

For more information about variable packing, see [here](https://blog.techfund.jp/p/solidity-variable-packing-saving-deployment-costs/#:~:text=Solidity%20variable%20packing%20is%20a,bytes%20equal%20to%20256%20bits.).

Instances: 1
- [src/RNSourceController.sol#L10](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L10)

&nbsp;
## [G-07] Use `private` rather than `public` for constants

Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table. If needed to be viewed externally, the values can be read from the verified contract source code.

Instances: 9
- [src/PercentageMath.sol#L8](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L8)
- [src/PercentageMath.sol#L11](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L11)
- [src/LotteryToken.sol#L12](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L12)
- [src/LotteryMath.sol#L14](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L14)
- [src/LotteryMath.sol#L16](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L16)
- [src/LotteryMath.sol#L18](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L18)
- [src/LotteryMath.sol#L20](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L20)
- [src/LotteryMath.sol#L22](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L22)
- [src/LotteryMath.sol#L24](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L24)

&nbsp;
## [G-08] Change `public` functions to `external`

Functions marked as `public` that are not called internally should be set to `external` to save gas and improve code quality. External call cost is less expensive than of public functions.

Instances: 2
- [src/LotterySetup.sol#L120](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120)
- [src/Lottery.sol#L235](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L235)

&nbsp;
## [G-09] Use a more recent version of Solidity

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than `revert()`/`require()` strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

Use a Solidity version of at least 0.8.12 to get string.concat() to be used instead of abi.encodePacked(<str>,<str>).

Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions.

Instances: 2
- [src/VRFv2RNSource.sol#L3](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L3)
- [src/interfaces/IVRFv2RNSource.sol#L3](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L3)

&nbsp;
## [G-10] Naming a return variable and still calling the `return` keyword wastes gas

Instances: 11
- [src/LotterySetup.sol#L120](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120)
- [src/PercentageMath.sol#L17](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L17)
- [src/PercentageMath.sol#L22](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L22)
- [src/Lottery.sol#L235](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L235)
- [src/Lottery.sol#L239](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L239)
- [src/ReferralSystem.sol#L115](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L115)
- [src/ReferralSystem.sol#L156](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L156)
- [src/ReferralSystem.sol#L161](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L161)
- [src/TicketUtils.sol#L24](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L24)
- [src/staking/Staking.sol#L48](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L48)
- [src/staking/Staking.sol#L61](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L61)

&nbsp;
## [G-11] `x += y` costs more gas than `x = x + y` for state variables

Instances: 3
- [src/Lottery.sol#L276](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L276)
- [src/staking/StakedTokenLock.sol#L30](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30)
- [src/staking/StakedTokenLock.sol#L43](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L43)

&nbsp;
## [G-12] Usage of `uint` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Consider using a larger size then downcasting where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

Instances: 17
- [src/LotterySetup.sol#L30](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L30)
- [src/LotterySetup.sol#L31](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L31)
- [src/Lottery.sol#L27](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L27)
- [src/Lottery.sol#L34](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L34)
- [src/Lottery.sol#L36](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L36)
- [src/Lottery.sol#L37](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L37)
- [src/Lottery.sol#L39](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L39)
- [src/ReferralSystem.sol#L17](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L17)
- [src/ReferralSystem.sol#L19](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L19)
- [src/ReferralSystem.sol#L21](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L21)
- [src/ReferralSystem.sol#L23](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L23)
- [src/ReferralSystem.sol#L25](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L25)
- [src/VRFv2RNSource.sol#L10](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L10)
- [src/VRFv2RNSource.sol#L11](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L11)
- [src/LotteryMath.sol#L24](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L24)
- [src/interfaces/ILotterySetup.sol#L71](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L71)
- [src/interfaces/ILotterySetup.sol#L73](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L73)

&nbsp;
## [G-13] `++i` costs less gas than `i++`

True for `--i`/`i--` as well, and is especially important in for loops. Saves 5 gas per iteration.

Instances: 7
- [src/Ticket.sol#L24](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L24)
- [src/Lottery.sol#L194](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L194)
- [src/Lottery.sol#L195](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L195)
- [src/Lottery.sol#L206](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L206)
- [src/Lottery.sol#L267](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L267)
- [src/TicketUtils.sol#L60](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L60)
- [src/TicketUtils.sol#L70](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L70)

&nbsp;
