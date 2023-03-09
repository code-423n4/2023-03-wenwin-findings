(1) We can pack structs to smaller size to save storage slot

Structs containing `uint256` should be checked again if it is really necessary to use `uint256` and instead can use a lower value such as `uint96` as it is unlikely there is ever need to pass the max value of `uint96`. By using `uint96` it can save significantly a large margin of gas to all affected structs.
Each storage slot would save about `~2.1k` in gas. Just by changing all `uint256` to `uint96` we can save about `~8` storage slots which total up to `~16,800` in gas savings. 

```solidity
ILotterySetup.sol:
   53: struct LotteryDrawSchedule {
   54:     /// @dev First draw is scheduled to take place at this timestamp
   55:     uint256 firstDrawScheduledAt;
   56:     /// @dev Period for running lottery
   57:     uint256 drawPeriod;
   58:     /// @dev Cooldown period when users cannot register tickets for draw anymore
   59:     uint256 drawCoolDownPeriod;
   60: }
   //@audit Can save about 2 storage slots


   63: struct LotterySetupParams {
   64:     /// @dev Token to be used as reward token for the lottery
   65:     IERC20 token;
   66:     /// @dev Parameters of the draw schedule for the lottery
   67:     LotteryDrawSchedule drawSchedule;
   68:     /// @dev Price to pay for playing single game (including fee)
   69:     uint256 ticketPrice;
   70:     /// @dev Count of numbers user picks for the ticket
   71:     uint8 selectionSize;
   72:     /// @dev Max number user can pick
   73:     uint8 selectionMax;
   74:     /// @dev Expected payout for one ticket, expressed in `rewardToken`
   75:     uint256 expectedPayout;
   76:     /// @dev Array of fixed rewards per each non jackpot win
   77:     uint256[] fixedRewards;
   78: }
   //@audit Can save about 2 storage slots


IReferralSystemDynamic.sol:
   24: struct MinimumReferralsRequirement {
   25:     uint256 minimumTicketsSold;
   26:     uint256 factor;
   27:     ReferralRequirementFactorType factorType;
   28: }
   //@audit Can save 1 storage slots


IRNSource.sol:
   39:     struct RandomnessRequest {
   40:         /// @dev specifies the request status
   41:         RequestStatus status;
   42:         /// @dev Random number generated for particular request
   43:         uint256 randomNumber;
   44:     }
   //@audit Can save about 1 storage slots, According to solidity docs: https://docs.soliditylang.org/en/v0.5.3/types.html#enums enums are of type `uint8`, changing `randomNumber` to `uint224` can save 1 storage slot.


(2) Significantly optimize gas usage by upgrading solidity version

The solidity version that is used on these contracts are significantly low: `0.8.7`. There has been many optimizations to the latest solidity version of `0.8.19` this can significantly boost up the gas saving of each contract that is used.

```solidity
IVRFv2RNSource.sol:
    3: pragma solidity ^0.8.7;

VRFv2RNSource.sol:
    3: pragma solidity ^0.8.7;

StakedTokenLock.sol:
    3: pragma solidity ^0.8.17;
```