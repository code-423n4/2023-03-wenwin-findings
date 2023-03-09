## [L-01] ****Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256****

The LotterySetup contract includes a function called **`packFixedRewards`**, which takes an argument called **`rewards`**. The **`rewards`** argument is an array of type **`uint256`**.

When the function calculates the reward, it divides **`rewards[winTier]`** by **`divisor`** and downcasts the result to **`uint16`**.

```solidity
File: src\LotterySetup.sol

function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed) {
						...
            uint16 reward = uint16(rewards[winTier] / divisor);
            ...
    }
```

**Recommended Mitigation Steps:**

Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256.

## [L-02] All functions in the library PercentageMath can revert with overflow

Since this library was developed to use on uint256 but in the function's body we have uin256 multiplication which potentially can overflow. Consider creating a helper contract with functions that have input validation or function params that cant revert with overflow after multiplication eg  

```solidity
function getPercentage(uint128 number, uint128 percentage) internal pure returns (uint256 result) {
        return number * percentage / PERCENTAGE_BASE;
    }
```

### [L-03] **`_safeMint()` should be used rather than `_mint()`**

The use of **`_mint()`** function is discouraged and it is recommended to use **`_safeMint()`**
 function instead. **`_safeMint()`** ensures that the recipient is either an EOA or implements the **`IERC721Receiver`** interface, making it a more secure option for minting new tokens.

```solidity
File: src\Ticket.sol

function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
        ticketId = nextTicketId++;
        ticketsInfo[ticketId] = TicketInfo(drawId, combination, false);
        _mint(to, ticketId);
    }
```

### [L-04] **Missing checks for `address(0x0)` when assigning values to `address` state variables**

Although this check is present in other contracts.

```solidity
File: src\staking\StakedTokenLock.sol

IStaking public immutable override stakedToken;
IERC20 public immutable override rewardsToken;
...

constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
    ...
    stakedToken = IStaking(_stakedToken);
    rewardsToken = stakedToken.rewardsToken();
		...
}
```

```solidity
File: src\RNSourceBase.sol

address internal immutable authorizedConsumer;
...
constructor(address _authorizedConsumer) {
    authorizedConsumer = _authorizedConsumer;
}
```

## [Q-01] Remove unused interfaces

`IReferralSystemDynamic.sol` does not seem to be used anywhere, as there are no imports or contracts that inherit from it.

## [Q-02] Don’t u**se the latest Solidity version unless it’s necessary**

It is generally not recommended to use the latest version of Solidity unless it is necessary, because new versions may contain undiscovered bugs or other issues that could affect the functionality or security of your smart contract. Instead, it is recommended to use a stable and well-tested version of Solidity that is known to be reliable and secure.