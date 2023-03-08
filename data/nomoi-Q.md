## 1. Bad naming
 
The `UnauthorizedClaim` error is [thrown by the `onlyTicketOwner` modifier](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L71) if `msg.sender` is not the ticket owner. However, if the `onlyTicketOwner` modifier were to be used in some function unrelated to claiming, the `UnauthorizedClaim` error would not make sense, defeating the purpose of encapsulating such logic in a reusable modifier. Consider using a more descriptive error name in this case.

## 2. Inconsistent naming

The [`whenNotExecutingDraw` and `onlyWhenExecutingDraw` modifiers](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L52-L65) are tightly related but are clearly named inconsistently (using the `only` prefix vs not using it) and it is not clear why such a difference is needed. For simplicity, consider removing the `only` prefix from (or adding it to) both modifiers.

## 3. Stricter validation

In the [`buyTickets` function](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L110) it is checked that `drawIds` and `tickets` are the same length, but it is not explicitly checked that they are not empty. This doesn't result in any vulnerability, as in such case the state is not changed because of checks performed in a later call to [`referralRegisterTickets `](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L62-L66), but this delegation of responsibility however makes the code harder to analyze and is prone to errors. Consider explicitly checking that the provided arrays are not empty.

## 4. Incorrect comment

The comment [`// Must be called in the deriving contract's `requestRandomnessFromUnderlyingSource` function`](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceBase.sol#L32) is wrong, as the `fulfill` function should be called when the randomness request is being fulfilled, not requested.

## 5. Incorrect uint size for ticket

The [`isValidTicket` function](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L18) receives a `uint256 ticket` as parameter. However, the rest of the codebase uses `uint120` for representing tickets. This doesn't result in any exploitable issue but is inconsistent and makes the code harder to reason about. Consider using the same `uint` type everywhere a ticket is being represented.

## 6. Inconsistent use of interfaces

In general, the codebase makes a good use of interfaces by only including a restricted set of function definitions that implementations must then implement. However, there are cases such as [`IRNSource.sol`](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/interfaces/IRNSource.sol#L33-L44) in which some things that only correspond to specific implementations are used (`RandomnessRequest` and `RequestStatus`). Consider only including implementation-independent definitions in interfaces.