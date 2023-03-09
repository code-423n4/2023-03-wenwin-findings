- [Low](#low)
    - [**1. Integer overflow by unsafe casting**](#1-integer-overflow-by-unsafe-casting)
    - [**2. Lack of checks address0**](#2-lack-of-checks-address0)
    - [**3. Lack of checks supportsInterface**](#3-lack-of-checks-supportsinterface)
    - [**4. Lack of checks integer ranges**](#4-lack-of-checks-integer-ranges)
    - [**5. Incompatible with tokens without decimals**](#5-incompatible-with-tokens-without-decimals)
    - [**6. The referer should never be yourself**](#6-the-referer-should-never-be-yourself)
    - [**7. Centralized risk**](#7-centralized-risk)
- [Non critical](#non-critical)
    - [**8. Unify return criteria**](#8-unify-return-criteria)
    - [**9. Improve naming**](#9-improve-naming)

# Low

## **1. Integer overflow by unsafe casting**

Keep in mind that the version of solidity used, despite being greater than `0.8`, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

**Recommendation:**

Use a [safeCast](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) from Open Zeppelin.

```javascript
    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
        return number * int256(percentage) / int256(PERCENTAGE_BASE);
    }
```

**Affected source code:**

- [PercentageMath.sol:23](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/PercentageMath.sol#L23)
- [LotteryMath.sol:48](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L48)
- [LotteryMath.sol:55](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L55)
- [LotteryMath.sol:64-65](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L64-L65)
- [ReferralSystem.sol:69-71](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L69-L71)

## **2. Lack of checks `address(0)`**

The following methods have a lack of checks if the received argument is an address, it's good practice in order to reduce human error to check that the address specified in the constructor or initialize is different than `address(0)`.

**Affected source code:**

- [RNSourceBase.sol:12](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceBase.sol#L12)

## **3. Lack of checks `supportsInterface`**

The `EIP-165` standard helps detect that a smart contract implements the expected logic, prevents human error when configuring smart contract bindings, so it is recommended to check that the received argument is a contract and supports the expected interface.

**Reference:**

- https://eips.ethereum.org/EIPS/eip-165

**Affected source code:**

- [RNSourceController.sol:78](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L78)
- [RNSourceController.sol:90](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L90)
- [StakedTokenLock.sol:18](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L18)
- [Staking.sol:31](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L31)

## **4. Lack of checks integer ranges**

The following methods lack checks on the following integer arguments, you can see the recommendations above.

**Affected source code:**

For example, funds deposited in `StakedTokenLock` could be locked forever with a bad initial configuration.

- [StakedTokenLock.sol:20-21](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L20-L21)

## **5. Incompatible with tokens without decimals**

Cannot set a `rewardToken` without decimals. Because the pragma used doesn't allow integer underflows, if the reward token used has 0 decimals, an integer underflow will produce a denial of service.

**Affected source code:**

- [LotterySetup.sol:168](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L168)
- [LotterySetup.sol:128](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L128)

## **6. The referer should never be yourself**

The `referrerTicketCount` counter of the same player can be increased, in order to increase its rewards in `claimPerDraw`, for this the `msg.sender` must be established as referrer.

**Affected source code:**

- [ReferralSystem.sol:60](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L60)

## **7. Centralized risk**

Nothing ensures that rewards tokens are available to give to the user, so if they are not transferred to the contract, the `getReward` method will be denied.

**Affected source code:**

- [Staking.sol:98](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/Staking.sol#L98)

---

# Non critical

## **8. Unify return criteria**

In `TicketUtils.sol` and `LotterySetup.sol` files, the methods are sometimes returned with `return`, other times the return variable is equal, and on other occasions the name of the return variable is not defined, It is recommended to unify the way of writing the code to make it more uniform and readable.

In the same file you can see different behaviors when programming returns. This behavior is observed across multiple contracts.

**Affected source code:**

- [TicketUtils.sol:24](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L24)
- [LotterySetup.sol:152-162](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L152-L162)

## **9. Improve naming**

Authoritative internals methods must use '`_`' before the name. We have an example in [Ticket.sol:19](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L19) where the methods that require authorization and it's internal, a different naming should be used.

**Affected source code:**

- [Ticket.sol:19](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L19)
- [Ticket.sol:23](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L23)
- [RNSourceBase.sol:33](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceBase.sol#L33)
- [RNSourceController.sol:38](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L38)

