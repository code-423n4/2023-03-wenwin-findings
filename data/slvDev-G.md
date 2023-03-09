|  | Issue | Instances |
| --- | --- | --- |
| [G-01] | Use constants instead of type(uintx).max | 1 |
| [G-02] | Using storage instead of memory for structs/arrays saves gas | 2 |
| [G-03] | Using calldata in function args saves gas | 1 |
| [G-04] | Use nested if and, avoid multiple check combinations | 2 |
| [G-05] | Functions guaranteed to revert_ when called by normal users can be marked payable | 6 |
| [G-06] | Setting the constructor to payable | 10 |
| [G-07] | x += y (x -= y) costs more gas than x = x + y (x = x - y) for state variables | 2 |
| [G-08] | Add unchecked {} for subtractions where the operands cannot underflow because of a previous require or if statement | 1 |
| [G-09] | internal functions only called once can be inlined to save gas | 3 |

### [G-01] Use constants instead of type(uintx).max

Using type(uint16).max and similar operations consumes more gas during both the distribution process and for each transaction compared to using constants.

```solidity
File: src\LotterySetup.sol

function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
        ...
            uint256 mask = uint256(type(uint16).max) << (winTier * 16);
        ...
    }
```

### [G-02] **Using `storage` instead of `memory` for `structs/arrays` saves gas**

When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declaring the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

```solidity
File: src\ReferralSystem.sol

function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {
        ... 
        UnclaimedTicketsData memory _unclaimedTickets = unclaimedTickets[drawId][msg.sender];
        ...
    }
```

```solidity
File: src\Lottery.sol

function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier) {
        TicketInfo memory ticketInfo = ticketsInfo[ticketId];
        ...
    }
```

### [G-03] Using `calldata` in function args saves gas

When a function argument is declared with the **`calldata`** keyword, the function can access the argument's data directly from the transaction's input data, which is already stored in the contract's memory.

```solidity
File: src\ReferralSystem.sol

function claimReferralReward(uint128[] memory drawIds) external override returns (uint256 claimedReward) 
```

### **[G-04] Use nested `if and`, avoid multiple check combinations**

**Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.**

```solidity
File: src\staking\StakedTokenLock.sol

function withdraw(uint256 amount) external override onlyOwner {
        ...
        if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration)
        ...
    }
```

```solidity
File: src\staking\StakedTokenLock.sol

function withdraw(uint256 amount) external override onlyOwner {
        ...
        if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {
        ...
    }
```

### [G-05] **Functions guaranteed to revert_ when called by normal users can be marked `payable`**

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

```solidity
File: src\staking\StakedTokenLock.sol

function deposit(uint256 amount) external override onlyOwner 
function withdraw(uint256 amount) external override onlyOwner 
```

```solidity
File: src\RNSourceController.sol
function initSource(IRNSource rnSource) external override onlyOwner 
function swapSource(IRNSource newSource) external override onlyOwner 
```

### [G-06] **Setting the *constructor* to `payable`**

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of `msg.value == 0` and saves `13 gas` on deployment with no security risks.

```solidity
File: src/LotteryToken.sol
constructor() ERC20("Wenwin Lottery", "LOT") 

File: src/VRFv2RNSource.sol	
constructor(address _authorizedConsumer, address _linkAddress, address _wrapperAddress, uint16 _requestConfirmations, uint32 _callbackGasLimit)

File: src/staking/StakedTokenLock.sol
constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration)

File: src/staking/Staking.sol
constructor(ILottery _lottery, IERC20 _rewardsToken, IERC20 _stakingToken, string memory name, string memory symbol) ERC20(name, symbol)

File: src/LotterySetup.sol
constructor(LotterySetupParams memory lotterySetupParams)

File: src/Lottery.sol
constructor(LotterySetupParams memory lotterySetupParams,uint256 playerRewardFirstDraw,uint256 playerRewardDecreasePerDraw,uint256[] memory rewardsToReferrersPerDraw,uint256 maxRNFailedAttempts,uint256 maxRNRequestDelay)	

File: src/Ticket.sol
constructor() ERC721("Wenwin Lottery Ticket", "WLT")

File: src/RNSourceBase.sol
constructor(address _authorizedConsumer) 

File: src/RNSourceController.sol
constructor(uint256 _maxFailedAttempts, uint256 _maxRequestDelay)

File: src/ReferralSystem.sol
constructor(uint256 _playerRewardFirstDraw,uint256 _playerRewardDecreasePerDraw, uint256[] memory _rewardsToReferrersPerDraw)
```

### **[G-07] `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables**

```solidity
File: src\staking\StakedTokenLock.sol

function deposit(uint256 amount) external override onlyOwner {
		...
    depositedBalance += amount;
		...
}

function withdraw(uint256 amount) external override onlyOwner {
    ...
    depositedBalance -= amount;
		...
}
```

### [G-08] Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require` or `if` statement

This will stop the check for overflow and underflow so it will save gas.

```solidity
function returnUnclaimedJackpotToThePot() private {
      if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {
          uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
          ...
		}
}
```

### [G-09] `internal` **functions only called once can be inlined to save gas**

```solidity
File: src\Lottery.sol

function requireFinishedDraw(uint128 drawId) internal view override 
function mintNativeTokens(address mintTo, uint256 amount) internal override 
```

```solidity
File: src\LotterySetup.sol

function _baseJackpot(uint256 _initialPot) internal view returns (uint256)
```