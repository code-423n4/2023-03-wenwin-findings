# Low

### `LotteryMath`: Hardcoded `DRAWS_PER_YEAR` may be incorrect

The `LotteryMath` helper assumes a hardcoded value of 52 `DRAWS_PER_YEAR`, equivalent to one draw per week:

[`LotteryMath#DRAWS_PER_YEAR`](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryMath.sol#L23-L25):

```solidity
    /// @dev Number of lottery draws per year
    uint128 public constant DRAWS_PER_YEAR = 52;

```

However, the draw period and schedule parameters are configurable in the `LotterySetup` contract, and it's possible to create a lottery with more or less than 52 draws per year.

This affects unclaimed ticket deadlines: unclaimed jackpots may be returned to the pot after 52 _draw periods_, not necessarily 52 weeks:

[`Lottery#returnUnclaimedJackpotToThePot`](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L271-L277)

```solidity
    function returnUnclaimedJackpotToThePot() private {
        if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {
            uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
            uint256 unclaimedJackpotTickets = unclaimedCount[drawId][winningTicket[drawId]];
            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
        }
    }
```

If a lottery is configured with frequent draws, the timeline to return unclaimed payouts may be shorter than expected.

### `StakedTokenLock`: Deposit deadline and lock duration are not validated

The `StakedTokenLock` contract is designed as a lock for Wenwin team tokens, to be "pre-staked and locked for 1 year" according to Wenwin [docs](https://docs.wenwin.com/wenwin-lottery/protocol-architecture/token).

However, the 1 year `lockDuration` and `depositDeadline` parameters are passed as constructor parameters and are not validated in the lock contract:

[`StakedTokenLock`](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/staking/StakedTokenLock.sol#L16-L22):

```solidity
    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
        _transferOwnership(msg.sender);
        stakedToken = IStaking(_stakedToken);
        rewardsToken = stakedToken.rewardsToken();
        depositDeadline = _depositDeadline;
        lockDuration = _lockDuration;
    }
```

Although these parameters would be publicly visible after deployment, the Wenwin team can deploy this contract with a zero lock duration. Consider validating that lock duration is at least 1 year.

### `Ticket`: Missing ERC721 metadata

The `Ticket` ERC721 does not implement a meaningful `tokenURI` function returning token metadata. This is not required my the ERC721 standard, but since Wenwin tickets are expected to trade on secondary markets, it's a good idea to implement meaningful metadata for your use case to mitigate secondary market scams. 

In particular, token metadata should show whether a ticket's draw is completed, whether the ticket is a winner, and whether or not the ticket's payout has already been claimed. Although it's possible to check all this data on chain, exposing it to secondary marketplaces through token metadata will help prevent scams on the secondary market.

## NC

### `Lottery`: Use new named mapping parameter syntax

Since these contracts are using Solidity >0.8.18, it's possible to use [named mapping parameters](https://blog.soliditylang.org/2023/02/01/solidity-0.8.18-release-announcement/). Consider using these to enhance readability of nested mappings:

[`Lottery.sol`](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L26-L27)

```solidity
    mapping(address => uint256) private frontendDueTicketSales;
    mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
```

Suggested:

```solidity
    mapping(address frontend => uint256 totalSales) private frontendDueTicketSales;
    mapping(uint128 draw => mapping(uint120 ticket => uint256 unclaimed)) private unclaimedCount;
```

### `LotteryToken`: Unused import
[`LotteryToken.sol`](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotteryToken.sol#L7-L8) imports but does not use `LotteryMath.sol`:

```solidity
import "src/LotteryMath.sol";
```

### `TicketUtils`: Comment typo

There's a small comment typo in [`TicketUtils`](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L16-L17):

```solidity
    /// @return isValid Is ticked valid
```