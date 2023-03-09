
# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | `fulfillRandomWords` should never revert | Low | 1 |
| 2 | randomness should not be requested multiple times | Low | 1 |
| 3 | Immutable state variables lack zero address checks | Low | 2 |
| 4 | `require` should be used instead of `assert` | NC | 2  |
| 5 | Named return variables not used anywhere in the functions | NC | 5 |

## Findings

### 1- `fulfillRandomWords` should never revert :

#### Risk : Low

The function `fulfillRandomWords` (in the `VRFv2RNSource`) contract should never revert according to the Chainlink [documentation](https://docs.chain.link/vrf/v2/security#fulfillrandomwords-must-not-revert), as in that case the VRF service will not attempt to call it a second time.

Further more the check statement in the `fulfillRandomWords` function seems to be of no effect as the actual randomness call to the function `requestRandomness` specify in the given arguments that only one random word is requested, so the check should be removed.

#### Proof of Concept

This issue occurs in the following instance : 

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L32-L38

#### Mitigation

The check statement inside the `fulfillRandomWords` should be removed to avoid the revert of the call.


### 2- randomness should not be requested multiple times:

#### Risk : Low

In the RNSourceController contract a logic is implemented to re-request randomness from the Chainlink VRF in case the first calls failed but this is warned againest in the Chainlink [documentation](https://docs.chain.link/vrf/v2/security#do-not-re-request-randomness) and can potentially introduce an attack surfaces that can impact the protocol working.

This issue should be reviewed by the protocol devs to make sure that their concepts work perfectely and does not have weaknesses or attacking areas.


### 3- Immutable state variables lack zero address checks  :

Constructors should check the values written in an immutable state variables(address) is not the zero address (address(0))

#### Risk : Low

#### Proof of Concept
Instances include:

File: StakedTokenLock.sol [Line 18](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L18)
```
stakedToken = IStaking(_stakedToken);
```

File: RNSourceBase.sol [Line 12](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L12)
```
authorizedConsumer = _authorizedConsumer;
```

#### Mitigation
Add non-zero address checks in the constructors for the instances aforementioned.

### 4- `require` should be used instead of `assert` :

Prior to solidity version 0.8.0, hitting an assert consumes the remainder of the transaction’s available gas rather than returning it, as `require()/revert()` do. `assert()` should be avoided even past solidity version 0.8.0 as its [documentation](https://docs.soliditylang.org/en/v0.8.14/control-structures.html#panic-via-assert-and-error-via-require) states that “The assert function creates an error of type Panic(uint256). … Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix”.

#### Risk : Non critical

#### Proof of Concept
Instances include:

File: LotterySetup.sol [Line 147](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147)
```
assert(initialPot > 0);
```

File: TicketUtils.sol [Line 99](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99)
```
assert((winTier <= selectionSize) && (intersection == uint256(0)));
```

#### Mitigation
Replace the `assert` checks aforementioned with `require` statements.


### 5- Named return variables not used anywhere in the function :

When Named return variable are declared they should be used inside the function instead of the return statement or if not they should be removed to avoid confusion.

#### Risk : Non critical

#### Proof of Concept
Instances include:

File: LotterySetup.sol [Line 120](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120)
```
returns (uint256 amount)
```

File: ReferralSystem.sol [Line 115](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L115)
```
returns (uint256 minimumEligible)
```

File: ReferralSystem.sol [Line 156](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L156)
```
returns (uint256 rewards)
```

File: ReferralSystem.sol [Line 161](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L161)
```
returns (uint256 rewards)
```

File: TicketUtils.sol [Line 24](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L24)
```
returns (bool isValid)
```

#### Mitigation

Either use the named return variables inplace of the return statement or remove them.