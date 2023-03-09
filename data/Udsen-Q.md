## 1. Check for `claimedAmount != 0`, BECAUSE SOME ERC20 REVERT ON ZERO TRANSFER

Some `erc20` tokens revert on zero transfer, hence it is recommended to check the transfer amount for zero value before calling the `safeTransfer`.

        rewardToken.safeTransfer(beneficiary, claimedAmount);
		
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L156
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L175

## 2. CHECK CONTRACT BALANCE IS GREATER THAN THE TRANSFERRING AMOUNT BEFORE THE TRANSFER FUNCTION IS CALLED

Currently code snippet looks like below, which does not check the token balance amount of contract against the actual transfer amount.

        claimedAmount = LotteryMath.calculateRewards(ticketPrice, dueTicketsSoldAndReset(beneficiary), rewardType);

        emit ClaimedRewards(beneficiary, claimedAmount, rewardType);
        rewardToken.safeTransfer(beneficiary, claimedAmount); 

Conduct the following check to make sure contract balance is greater than the transfer amount before transfering the `claimedAmount` to the `beneficiary`.

	require(rewardToken.balanceOf(address(this)) >= claimedAmount, "Not enough funds to transfer");
	
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L151-L157
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L170-L176


## 3. `delete` THE VARIABLE WHEN RESETTING IT, INSTEAD OF ASSIGNING THE VALUE TO ZERO

            frontendDueTicketSales[beneficiary] = 0;

It is recommended to use `delete` to reset the variable instead of assigning zero value to the variable.

            delete frontendDueTicketSales[beneficiary];
			
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L255
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L95

## 4. USE `safeCast` WHEN CONVERTING `uint256` VALUE TO `int256`

When casting from uint256 to int256 higher order bits could get excluded.
Hence it is recommended to use `openzeppelin` `SafeCast.sol` library and use its  `toInt256(uint256 value)` function to perform safeCast of the `uint256` variable to a `int256` variable.

            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
            
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L64

## 5. EXPRESSIONS AND MATHEMATICAL CALCULATIONS SHOULD NOT BE DECLARED AS CONSTANTS BUT RATHER AS IMMUTABLE VARIABLES

Variables and mathematical calculations should not be declared as `constants`. They should be declared as `immutable` variables and perform the calculations and assign the resulting values inside the `constructor`.

    /// @dev percentage of ticket price being paid for staking reward
    uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;
    /// @dev percentage of ticket price being paid to frontend operator
    uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;
    /// @dev Percentage of the ticket price that goes to the pot
    uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;
    /// @dev safety margin used to calculate excess pot, in percentage
    uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;
    /// @dev Percentage of excess pot reserved for bonus
    uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;
    /// @dev Number of lottery draws per year
    uint128 public constant DRAWS_PER_YEAR = 52;
	
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L13-L24

## 6. DO NOT USE MAGIC NUMBERS, USE `10**` TO DEPICT NUMBERS FOR BETTER READABILITY OF THE CODE.

    uint256 public constant override INITIAL_SUPPLY = 1_000_000_000e18;
	
Above line of code can be rewritten as follows for better code readability.

    uint256 public constant override INITIAL_SUPPLY = 10**27;	

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L12

## 7. OWNER OF THE `LotteryToken.sol` CONTRACT IS IMMUTABLE. HENCE CAN NOT BE CHANGED UNDER ANY CIRCUMSTANCE WHICH CREATES HIGH CENTRALIZATION RISK

The `owner` of the `LotteryToken.sol` is an immutable variable which is set inside the constructor. Hence it can not be changed under any circumstance. 
If the `owner` becomes a malicious user or gets compramised, it will create a higher centralization risk to the protocol since `LotteryToken` is the main `erc20` token of the protocol.

It is recommended to use a multisig wallet as the owner and have a two step ownership transfer procedure to change the owner incase of an emergency.

    address public immutable override owner;

    /// @dev Initializes lottery token with `INITIAL_SUPPLY` pre-minted tokens
    constructor() ERC20("Wenwin Lottery", "LOT") {
        owner = msg.sender;
        _mint(msg.sender, INITIAL_SUPPLY);
    }

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L14-L20	
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L12

## 8. DO NOT USE THE PUBLIC INTERFACE OF THE CONTRACT TO CALL FUNCTIONS WHEN THE CONTRACT IS INHERITED

No need to use the public interface of the contract to call a function in it, when the contract is inherited and is a parent contract.

`ITicket.sol` is an inherited interface to the `Ticket.sol` contract. Still the `ITicket` public interface is used to call the `TicketInfo` struct in the interface as shown below:

    mapping(uint256 => ITicket.TicketInfo) public override ticketsInfo;

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L14

## 9. USE `require` INSTEAD OF `assert` TO CHECK FOR CONDITONS

`assert` is used to uphold the invariants of a program which should not be broken under any condition.
To check for conditions which can be broken under user inputs and program variants, it is recommended to use `require` instead.

            assert((winTier <= selectionSize) && (intersection == uint256(0)));
			
Above statement can be replaced as follows:

            require((winTier <= selectionSize) && (intersection == uint256(0)));
			
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99

## 10. CHECK FOR `rewardPerToken() > userRewardPerTokenPaid[account]` CONDTION TO VERIFY NO UNDERFLOW WOULD OCCUR

In the `earned()` function of the `Staking.sol` the following calculation is performed to calculate the earned rewards for an user account.

        return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];
		
Here prior condition is required, to check `rewardPerToken() > userRewardPerTokenPaid[account]` to avoid any revert which would happen due to underflow.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L62

## 11. BETTER TO CALCULATE THE ACTUAL TRANSFERRED TOKEN AMOUNT (SINCE THERE COULD BE TRANSFER FEE) TO DETERMINE THE NUMBER OF SHARES TO MINT

In the `stake()` function of the `Stake.sol` contract, the `amount` reffered to in the `safeTransferFrom` function is used as the number of shares to mint.
The following code snippet depicts it.

        stakingToken.safeTransferFrom(msg.sender, address(this), amount);
        _mint(msg.sender, amount);
		
There are certain `erc20` tokens which charges tranfer fee during token transfers from one account to another. If the `stakingToken` happens to charge a transfer fee then the minted shares to the staker will be more than what the staker staked in the contract.
So when the staker withdraws the staked tokens by redeeming the shares, the contract will have to send more tokens than what was actually staked (transferred tokens) by the staker thus stealing funds from other stakers(depositers).

should calculate the before transfer and after transfer token balance of the contract and then get the differnce between the two to calculate the actual transfered amount.
Then use that amount as the share amount to mint to the user.

The following code snippet depicts the recommended change to the code:

		uint256 balance_before = stakingToken.balanceOf(address(this))
            stakingToken.safeTransferFrom(msg.sender, address(this), amount);
		uint256 balance_after = stakingToken.balanceOf(address(this))
		uint256 transferedAmount = balance_after - balance_before;
            _mint(msg.sender, transferedAmount);	

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L73-L74
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L79-L89

## 12. USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’

Can import the required specific contracts, functions or variables by using the named imports explicitly. Plain imports will import the entire context of the imported contract which could lead into variable namespace conflicts etc ...

	import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
	import "@openzeppelin/contracts/utils/math/Math.sol";
	import "src/ReferralSystem.sol";
	import "src/RNSourceController.sol";
	import "src/staking/Staking.sol";
	import "src/LotterySetup.sol";
	import "src/TicketUtils.sol";

Currently the `RNSourceController` is imported as follows:
	import "src/RNSourceController.sol";

But it can be imported explicitly by the name as follows:
	import {RNSourceController} from "src/RNSourceController.sol";
	
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L5-L11

All contracts have this issue.

## 13. NATSPEC IS MISSING

NatSpec is missing for the following contracts, functions, constructor and modifier.

`Lottery.sol`, `RNSourceController.sol`, `Staking.sol`, `LotterySetup.sol`, `Ticket.sol`, `LotteryToken.sol` and `ReferralSystem.sol` are main contracts of the project. But NatSpec is either missing or half completed in the code. 

It is recommended to use the NatSpec for commenting for better readability and understanding of the code base by developers and auditors.