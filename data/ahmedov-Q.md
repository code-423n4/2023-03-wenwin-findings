# QA Report - Wenwin

| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 4 |
|:--:|:--:|

### Non-Critical Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | Create your own import names instead of using the regular ones | 36 |

| Total Non-Critical Issues | 1 |
|:--:|:--:|

### Refactor Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | Private and internal functions should start with underscore | 26 |
| [R-02] | Remove unused imports | 1 |
| [R-03] | Use require instead of assert | 2 |

| Total Refactor Issues | 3 |
|:--:|:--:|

### [N-01] Create your own import names instead of using the regular ones

For better readability, you should name the imports instead of using the regular ones.

Example:
```solidity
import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

### [R-01] Private and internal functions should start with underscore

```solidity    
src/VRFv2RNSource.sol

28: function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId);
32: function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override;

src/LotterySetup.sol

164: function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed);

src/Lottery.sol

181: function registerTicket(
        uint128 drawId,
        uint120 ticket,
        address frontend,
        address referrer
    )
        private
        beforeTicketRegistrationDeadline(drawId)
        requireValidTicket(ticket)
        returns (uint256 ticketId);

203: function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw;
238: function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize);
249: function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets);
259: function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount);
271: function returnUnclaimedJackpotToThePot() private;
279: function requireFinishedDraw(uint128 drawId) internal view override;
285: function mintNativeTokens(address mintTo, uint256 amount) internal override;

src/Ticket.sol

19: function markAsClaimed(uint256 ticketId) internal;
23: function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId);

src/RNSourceBase.sol

33: function fulfill(uint256 requestId, uint256 randomNumber) internal;
48: function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId);

src/RNSourceController.sol

38: function requestRandomNumber() internal;
58: function receiveRandomNumber(uint256 randomNumber) internal virtual;
106: function requestRandomNumberFromSource() private;

src/ReferralSystem.sol

52: function referralRegisterTickets(
        uint128 currentDraw,
        address referrer,
        address player,
        uint256 numberOfTickets
    )
        internal;

74: function mintNativeTokens(address mintTo, uint256 amount) internal virtual;
87: function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal;
111: function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
        internal
        view
        virtual
        returns (uint256 minimumEligible);
134: function requireFinishedDraw(uint128 drawId) internal view virtual;
136: function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward);
156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards);
161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards);
```

### [R-02] Remove unused imports

```solidity
src/LotteryToken.sol

7: import "src/LotteryMath.sol";
```

### [R-03] Use require instead of assert

The Solidity assert() function is meant to assert invariants. Properly functioning code should never reach a failing assert statement.

