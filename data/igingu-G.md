# [G-01] Constant values should be precomputed and saved as constants

```solidity
uint256(0)
uint256(type(uint16).max)
uint256(1)
uint120(1)
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L45
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L126

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L29
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L32
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L74
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99

Present in multiple other places as well.

## Recommendation:

```solidity
src/LotterySetup.sol:37:
    uint256 private constant UINT256_ZERO = 0;

src/LotterySetup.sol:38:
    uint256 private constant UINT256_MAX = type(uint256).max;


src/LotterySetup.sol:45-47:
    if (lotterySetupParams.ticketPrice == UINT256_ZERO) {
        revert TicketPriceZero();
    }

src/LotterySetup.sol:127:
    uint256 mask = UINT256_MAX << (winTier * 16);
```

# [G-02] Function parameters that don't mutate can be calldata

https://dev.to/jamiescript/gas-saving-techniques-in-solidity-324c#calldata

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L119

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L164

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L114
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L116

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L76

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L32

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L45

## Recommendation:

```solidity
src/Loterry.sol:119:
    returns (uint256[] memory ticketIds) -> returns (uint256[] calldata ticketIds)
```

Make sure to change in interfaces as well, where applicable.

# [G-03] Remove unnecessary duplicated assignment

```solidity
ReferralSystem.sol:139:
    UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
    ->
    UnclaimedTicketsData storage _unclaimedTickets = unclaimedTickets[drawId][msg.sender]; // saves gas

ReferralSystem.sol:145:
    // can be removed
    _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
```

# [G-04] Small `internal` functions only called once can be inlined to save gas

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L285

can be inlined directly into

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L81

# [G-05] RandomnessRequest can initialize only status at creation time, without `randomNumber: 0`, to save gas

```solidity
src/RNSourceBase:27:
    requests[requestId] = RandomnessRequest({ status: RequestStatus.Pending, randomNumber: 0 });
    ->
    requests[requestId].status = RequestStatus.Pending
```

# [G-06] Superfluous event fields

Emitting `msg.sender` is redundant and wastes gas, as this field is already included into any event's data.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L23

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSource.sol#L26

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol#L61
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol#L66
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol#L71

# [G-07] Copy struct from storage to memory, and only copy it to storage at the end

Reading and writting from/to storage multiple times is very gas costly. When using the same struct multiple times, it is
cheaper to copy it to memory and only store it at the end.

```solidity
function fulfill(uint256 requestId, uint256 randomNumber) internal {
    RandomnessRequest memory request = requests[requestId];
    if (request.status == RequestStatus.None) {
        revert RequestNotFound(requestId);
    }
    if (request.status == RequestStatus.Fulfilled) {
        revert RequestAlreadyFulfilled(requestId);
    }

    request.randomNumber = randomNumber;
    request.status = RequestStatus.Fulfilled;
    requests[requestId] = request;
    IRNSourceConsumer(authorizedConsumer).onRandomNumberFulfilled(randomNumber);

    emit RequestFulfilled(requestId, randomNumber);
}
```

# [G-08] `LotterySetupParams` can be slightly re-ordered to use one less storage slot and save gas

Inside structs, consecutive structure members can pack into the same storage slot, if their data sizes are <= 32 bytes.

More details here: https://fravoll.github.io/solidity-patterns/tight_variable_packing.html

We can pack the `IERC20 token` (20 bytes) + `uint8 selectionSize` (1 byte) + `uint8 selectionMax` (1 byte) by storing
them next to each other.

```solidity
// LotterySetupParams should change order to, so that token, selectionSize and selectionMax can pack into a single storage slot
struct LotterySetupParams {
    /// @dev Token to be used as reward token for the lottery
    IERC20 token;
    /// @dev Count of numbers user picks for the ticket
    uint8 selectionSize;
    /// @dev Max number user can pick
    uint8 selectionMax;
    /// @dev Parameters of the draw schedule for the lottery
    LotteryDrawSchedule drawSchedule;
    /// @dev Price to pay for playing single game (including fee)
    uint256 ticketPrice;
    /// @dev Expected payout for one ticket, expressed in `rewardToken`
    uint256 expectedPayout;
    /// @dev Array of fixed rewards per each non jackpot win
    uint256[] fixedRewards;
} can be packed to save on storage
```

# [G-09] TicketUtils reconstructTicket could benefit as well from unchecked blocks

The whole `reconstructTicket` function body could be put in an `unchecked {}` block as well, to save on gas. With the
assumption that selectionSize remains <= 16, none of the instructions in the function body can overflow.

# [G-10] Functions guaranteed to revert if called by users can be marked `payable` to save gas

When functions are not `payable`, Solidity compiler introduces 10 more opcodes, to assert that `msg.value` is 0 =>
CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2), which cost
about 21 gas per call to the function, in addition to extra deployment cost.

`onlyOwner` functions could be payable, as owners are less prone to send ETH to the contract by mistake.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L77
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L89

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L24
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L37

All constructors can also be made `payable`, to avoid additional gas cost on deployment.

The above is a saving with no security risk, other than owners accidentally sending ether to the smart contracts.

# [G-11] Use `assembly` to write address storage values

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L41
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L42
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L43

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L12

## Recommendation:

```solidity
src/RNSourceBase.sol:12:
    authorizedConsumer = _authorizedConsumer;
    ->
    assembly {
        sstore(authorizedConsumer.slot, _authorizedConsumer)
    }
```

# [G-12] `x += y` or `x -= y` costs more than `x = x + y` or `x = x - y` for state variables

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L129

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L64
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L67
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L69
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L71

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L43

# [G-13] Enabling Solidity's optimizer would save gas, especially for frequently called functions like `registerTickets`, `claimWinningTickets`, `claimRewards`

https://docs.soliditylang.org/en/v0.8.17/internals/optimizer.html#benefits-of-optimizing-solidity-code

Here is how to [enable it for Foundry's forge](https://book.getfoundry.sh/reference/forge/forge-build#the-optimizer).

Specifying a high number of iterations would increase deployment cost, but reduce call cost of frequently used functions
(`registerTickets`, `claimWinningTickets`, `claimRewards`, and more).
