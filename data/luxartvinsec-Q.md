# 1. Potential Reentrancy vulnerability
 
### Link : https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L91
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L132
https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/interfaces/ILotteryToken.sol#L22
https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Ticket.sol#L23
https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/LotteryToken.sol#L22
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L181

### Recommendation: 

One possible solution to fix the vulnerability is to use the "Checks-Effects-Interactions" pattern to ensure that external contract calls are executed last. Specifically, the contract can keep a boolean flag to indicate if an update is in progress and revert any function call that tries to perform an external contract call when the flag is set. The flag can be unset after updating the storage. Another option is to use the `ReentrancyGuard` pattern provided by OpenZeppelin to protect against reentrancy attacks.

### Example Fix:

Option 1: Implement Checks-Effects-Interactions Pattern

```
bool private _updatingReward = false;

function getReward() public override {
    _updateReward(msg.sender);
    uint256 reward = rewards[msg.sender];
    if (reward > 0) {
        rewards[msg.sender] = 0;
        uint256 currentRewardPerToken = rewardPerToken();
        rewardPerTokenStored = currentRewardPerToken;
        lastUpdateTicketId = lottery.nextTicketId();
        rewardsToken.safeTransfer(msg.sender, reward);
        emit RewardPaid(msg.sender, reward);
    }
}

function _updateReward(address account) internal {
    require(!_updatingReward, "Reward update in progress");
    uint256 currentRewardPerToken = rewardPerToken();
    rewardPerTokenStored = currentRewardPerToken;
    lastUpdateTicketId = lottery.nextTicketId();
    rewards[account] = earned(account);
    userRewardPerTokenPaid[account] = currentRewardPerToken;
    _updatingReward = false;
}
```

Option 2: Use `ReentrancyGuard`

```
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract Staking is IStaking, ERC20, ReentrancyGuard {
    ...
    function getReward() public override nonReentrant {
        _updateReward(msg.sender);
        uint256 reward = rewards[msg.sender];
        if (reward > 0) {
            rewards[msg.sender] = 0;
            lottery.claimRewards(LotteryRewardType.STAKING);
            rewardsToken.safeTransfer(msg.sender, reward);
            emit RewardPaid(msg.sender, reward);
        }
    }
    ...
}
```

# 2. Incorrect Comparison in `LotterySetup.sol`

### Link: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L104

Incorrect comparison: The function `requireJackpotInitialized` contains an incorrect comparison that can be bypassed by an attacker to access functions that should be blocked. The contract incorrectly checks if the `initialPot` is equal to 0, instead of checking if it is greater than 0. This issue allows an attacker to access functions that should only be accessible once the `initialPot` is raised.

Incorrect comparison: To fix the incorrect comparison issue, change the if statement in the `requireJackpotInitialized` function to check if the `initialPot` is greater than 0, instead of checking if it is equal to 0.

To fix the issue, change the `requireJackpotInitialized` function from:

```
modifier requireJackpotInitialized() { if (initialPot == 0) { revert JackpotNotInitialized(); } _; }
```

to:

```
modifier requireJackpotInitialized() { if (initialPot <= 0) { revert JackpotNotInitialized(); } _; }
```

# 3.  Use SafeMath for security

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L161

# 4. Vulnerability in Lottery Smart Contract

### Link : https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L110

### Summary: 

The Lottery Smart Contract has a vulnerability in the `buyTickets` function, allowing an attacker to exploit the lottery by buying a ticket without actually paying for it.

### Impact: 

An attacker can buy a ticket for free without actually paying the required ticket price, thereby manipulating the lottery and taking away the potential rewards from legitimate users.

### Recommendation: 

To fix this vulnerability, the smart contract should ensure that the buyer has paid the required ticket price before registering the ticket. This can be achieved by transferring the ticket price from the buyer's account to the smart contract's account before registering the ticket.

### Example:

In the `buyTickets` function, the smart contract should transfer the ticket price from the buyer's account to the smart contract's account before registering the ticket. The updated function would look like this:

```
function buyTickets(
    uint128[] calldata drawIds,
    uint120[] calldata tickets,
    address frontend,
    address referrer
)
    external
    override
    requireJackpotInitialized
    returns (uint256[] memory ticketIds)
{
    if (drawIds.length != tickets.length) {
        revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
    }
    ticketIds = new uint256[](tickets.length);
    uint256 totalTicketPrice = ticketPrice * tickets.length;
    rewardToken.safeTransferFrom(msg.sender, address(this), totalTicketPrice);
    for (uint256 i = 0; i < drawIds.length; ++i) {
        ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
    }
    referralRegisterTickets(currentDraw, referrer, msg.sender, tickets.length);
    frontendDueTicketSales[frontend] += tickets.length;
}
```

With this update, the smart contract ensures that the buyer has paid the required ticket price before registering the ticket, thereby preventing any exploitation of the lottery by attackers.
