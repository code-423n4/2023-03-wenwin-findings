| Total Low issues |
|------------------|

| Risk   | Issues Details                                                                     | Number        |
|--------|------------------------------------------------------------------------------------|---------------|
| [L-01] | Avoid shadowing inherited state variables                                          | 2             |
| [L-02] | Loss of precision due to rounding                                                  | 7             |
| [L-03] | Use `safeMint` instead of mint for ERC721                                          | 1             |
| [L-04] | Integer overflow by unsafe casting                                                 | 6             |
| [L-05] | Missing checks for `address(0)`                                                    | 2             |
| [L-06] | Multiple Pragma used                                                               | All Contracts |
| [L-07] | Use `require()` instead of `assert()`                                              | 2             |
| [L-08] | LotteryToken.sol dosen't have a transfer ownership pattern                         | 1             |

| Total Non-Critical issues |
|---------------------------|

| Risk    | Issues Details                                                                                        | Number        |
|---------|-------------------------------------------------------------------------------------------------------|---------------|
| [NC-01] | Include return parameters in NatSpec comments                                                         | All Contracts |
| [NC-02] | Non-usage of specific imports                                                                         | All Contracts |
| [NC-03] | Lack of event emit                                                                                    | 1             |
| [NC-04] | Function writing does not comply with the `Solidity Style Guide`                                      | All Contracts |
| [NC-05] | The protocol should include NatSpec                                                                   | All Contracts |
| [NC-06] | Constants in comparisons should appear on the left side                                               | 10            |
| [NC-07] | Use a more recent version of solidity                                                                 | 2             |
| [NC-08] | Generate perfect code headers every time                                                              | All Contracts |
| [NC-09] | Lock pragmas to specific compiler version                                                             | 2             |
| [NC-10] | There is no need to cast a variable that is already an uint, such as `uint(x)`                        | 3             |
| [NC-11] | Consider using `delete` rather than assigning zero to clear values                                    | 4             |
| [NC-12] | For functions, follow Solidity standard naming conventions                                            | All Contracts |
| [NC-13] | Add NatSpec Mapping comment                                                                           | 17            |
| [NC-14] | Constants should be defined rather than using magic numbers                                           | 4             |
| [NC-15] | Not using the named return variables anywhere in the function is confusing                            | 15            |

## [L-01] Avoid shadowing inherited state variables

#### Description

In `Staking.sol` there is a local variables named `name` `symbol`, but there is functions and state variables in the inherited contract ( `ERC20.sol`) with the same name. This use causes compilers to issue warnings, negatively affecting checking and code readability.

#### Lines of code

```solidity
        string memory name,
```

- [Staking.sol#L26](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L26)

```solidity
        string memory symbol
```

- [Staking.sol#L27](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L27)

#### Recommended Mitigation Steps

Avoid using variables with the same name.

## [L-02] Loss of precision due to rounding

#### Description

Loss of precision due to the nature of arithmetics and rounding errors.

#### Lines of code 

```solidity
        return rewardPerTokenStored + (unclaimedRewards * 1e18 / _totalSupply);
```

- [Staking.sol#L58](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L58)

```solidity
        return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];
```

- [Staking.sol#L62](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L62)

```solidity
            uint16 reward = uint16(rewards[winTier] / divisor);
```

- [LotterySetup.sol#L170](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L170)

```solidity
                referrerRewardForDraw / totalTicketsForReferrersPerCurrentDraw;
```

- [ReferralSystem.sol#L100](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L100)

```solidity
            playerRewardsPerDrawForOneTicket[drawFinalized] = playerRewardForDraw / ticketsSoldDuringDraw;
```

- [ReferralSystem.sol#L105](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L105)

```solidity
        return number * percentage / PERCENTAGE_BASE;
```

- [PercentageMath.sol#L18](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L18)

```solidity
        return number * int256(percentage) / int256(PERCENTAGE_BASE);
```

- [PercentageMath.sol#L23](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L23)

## [L-03] Use `safeMint` instead of mint for ERC721

#### Description

Users could lost their NFTs if `msg.sender` is a contract address that does not support `ERC721`, the NFT can be frozen in the contract forever.

As per the documentation of EIP-721:

> A wallet/broker/auction application MUST implement the wallet interface if it will accept safe transfers.

> Ref: https://eips.ethereum.org/EIPS/eip-721

As per the documentation of ERC721.sol by Openzeppelin:

> Ref: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L274-L285

#### Lines of code 

```solidity
        _mint(to, ticketId);
```

- [Ticket.sol#L26](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L26)

#### Recommended Mitigation Steps

Use [`_safeMint`](https://github.com/transmissions11/solmate/blob/main/src/tokens/ERC721.sol#L193-L202) instead of [`mint`](https://github.com/transmissions11/solmate/blob/main/src/tokens/ERC721.sol#L157-L170) to check received address support for ERC721 implementation.

## [L-04] Integer overflow by unsafe casting

#### Description

Keep in mind that the version of solidity used, despite being greater than 0.8, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

#### Lines of code 

```solidity
            uint16 reward = uint16(rewards[winTier] / divisor);
```

- [LotterySetup.sol#L170](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L170)

```solidity
        excessPot = excessPotInt > 0 ? uint256(excessPotInt) : 0;
```

- [LotteryMath.sol#L65](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L65)

```solidity
            numbers[i] = uint8(randomNumber % currentSelectionCount);
```

- [TicketUtils.sol#L58](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L58)

```solidity
                winTier += uint8(intersection & uint120(1));
```

- [TicketUtils.sol#L96](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L96)

```solidity
            unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);
```

- [ReferralSystem.sol#L69](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L69)

```solidity
        unclaimedTickets[currentDraw][player].playerTicketCount += uint128(numberOfTickets);
```

- [ReferralSystem.sol#L71](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L71)

#### Recommended Mitigation Steps

Use OpenZeppelin [safeCast](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) library.

## [L-05] Missing checks for `address(0)`

#### Description

Check of `address(0)` to protect the code from `(0x0000000000000000000000000000000000000000)` address problem just in case. This is best practice or instead of suggesting that they verify `_address != address(0)`, you could add some good NatSpec comments explaining what is valid and what is invalid and what are the implications of accidentally using an invalid address.

#### Lines of code 

```solidity
        authorizedConsumer = _authorizedConsumer;
```

- [RNSourceBase.sol#L12](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L12)

```solidity
        stakedToken = IStaking(_stakedToken);
```

- [StakedTokenLock.sol#L18](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L18)

#### Recommended Mitigation Steps

Add checks for `address(0)` when assigning values to address state variables.	

## [L-06] Multiple Pragma used 

#### Description

Solidity pragma versions should be exactly same in all contracts. Currently some contracts use `0.8.19` and the rest use `^0.8.17` and `^0.8.7`

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-wenwin/tree/main/src)

#### Recommended Mitigation Steps

Use one version for all contracts.

## [L-07] Use `require()` instead of `assert()`

#### Description

Assert should not be used except for tests, require should be used

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process's available gas instead of returning it, as `request()/revert()` did.

The big difference between the two is that the `assert()` function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a `require()` statement when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

Assertion should be avoided even after solidity version 0.8.0, because its documentation states "The Assert function generates an error of type `Panic(uint256)`. Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there's a mistake".

#### Lines of code 

```solidity
        assert(initialPot > 0);
```

- [LotterySetup.sol#L147](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L147)

```solidity
            assert((winTier <= selectionSize) && (intersection == uint256(0)));
```

- [TicketUtils.sol#L99](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99)

#### Recommended Mitigation Steps

Use `require()` instead of `assert()`

## [L-08] LotteryToken.sol dosen't have a transfer ownership pattern

#### Description

A transfer ownership pattern is the best practice in case the owner's private keys are lost.

#### Lines of code 

- [LotteryToken.sol](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol)

#### Recommended Mitigation Steps

Use OpenZeppelin [Ownable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol) to make the contracts more robust in the future.

## [NC-01] Include return parameters in NatSpec comments

#### Description

If Return parameters are declared, you must prefix them with `/// @return`.
Some code analysis programs do analysis by reading [NatSpec](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) details, if they can't see the `@return` tag, they do incomplete analysis.

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-wenwin/tree/main/src)

#### Recommended Mitigation Steps

Include `@return` parameters in NatSpec comments

## [NC-02] Non-usage of specific imports

#### Description

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace. Instead, the Solidity docs recommend specifying imported symbols explicitly. 

- https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-wenwin/tree/main/src)

#### Recommended Mitigation Steps

Use specific imports syntax per solidity docs recommendation.

## [NC-03] Lack of event emit

#### Description

The below methods do not emit an event when the state changes, something that it's very important for dApps and users.

#### Lines of code 

```solidity
    function claimWinningTickets(uint256[] calldata ticketIds) external override returns (uint256 claimedAmount) {
        uint256 totalTickets = ticketIds.length;
        for (uint256 i = 0; i < totalTickets; ++i) {
            claimedAmount += claimWinningTicket(ticketIds[i]);
        }
        rewardToken.safeTransfer(msg.sender, claimedAmount);
    }
```

- [Lottery.sol#L170](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L170)

#### Recommended Mitigation Steps

Emit event.

## [NC-04] Function writing does not comply with the `Solidity Style Guide`

#### Description

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Functions should be grouped according to their visibility and ordered:

- `constructor()`
- `receive()`  
- `fallback()`  
- `external / public / internal / private`
- `view / pure`

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-wenwin/tree/main/src)

#### Recommended Mitigation Steps

Follow Solidity Style Guide.

## [NC-05] The protocol should include NatSpec

#### Description

It is recommended that Solidity contracts are fully annotated using NatSpec, it is clearly stated in the Solidity official documentation.

- In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

- Some code analysis programs do analysis by reading NatSpec details, if they can't see the tags `(@param, @dev, @return)`, they do incomplete analysis.

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-wenwin/tree/main/src)

#### Recommended Mitigation Steps

Include [`NatSpec`](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) comments in the codebase.

## [NC-06] Constants in comparisons should appear on the left side

#### Description

Constants in comparisons should appear on the left side, doing so will prevent typo [bug](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html).

```solidity
        if (lotterySetupParams.selectionSize == 0) {
```

#### Lines of code 

```solidity
        if (claimedAmount == 0) {
```

- [Lottery.sol#L262](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L262)

```solidity
        if (lotterySetupParams.selectionSize == 0) {
```

- [LotterySetup.sol#L48](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L48)

```solidity
        if (initialPot == 0) {
```

- [LotterySetup.sol#L106](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L106)

```solidity
        } else if (winTier == 0 || winTier > selectionSize) {
```

- [LotterySetup.sol#L123](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L123)

```solidity
        if (_rewardsToReferrersPerDraw.length == 0) {
```

- [ReferralSystem.sol#L32](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L32)

```solidity
            if (_rewardsToReferrersPerDraw[i] == 0) {
```

- [ReferralSystem.sol#L36](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L36)

```solidity
        if (ticketsSoldDuringDraw == 0) {
```

- [ReferralSystem.sol#L89](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L89)

```solidity
        if (_totalSupply == 0) {
```

- [Staking.sol#L50](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L50)

```solidity
        if (amount == 0) {
```

- [Staking.sol#L69](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L69)

```solidity
        if (amount == 0) {
```

- [Staking.sol#L81](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L81)

#### Recommended Mitigation Steps

Constants should appear on the left side:

```solidity
        if (0 == lotterySetupParams.selectionSize) {
```

## [NC-07] Use a more recent version of solidity

#### Description

For security, it is best practice to use the [latest Solidity version](https://github.com/ethereum/solidity/blob/develop/Changelog.md).

#### Lines of code 

```solidity
pragma solidity ^0.8.17;
```

- [StakedTokenLock.sol#L3](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L3)

```solidity
pragma solidity ^0.8.7;
```

- [VRFv2RNSource.sol#L3](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L3)

#### Recommended Mitigation Steps

Old versions of Solidity are used `(^0.8.7)`, `(^0.8.17)`,  newer version can be used `(0.8.19)`.

## [NC-08] Generate perfect code headers every time

#### Description

I recommend using header for Solidity code layout and readability

> Ref: https://github.com/transmissions11/headers

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-wenwin/tree/main/src)

#### Recommended Mitigation Steps

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## [NC-09] Lock pragmas to specific compiler version

#### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally. 

> Ref: https://swcregistry.io/docs/SWC-103

#### Lines of code 

```solidity
pragma solidity ^0.8.17;
```

- [StakedTokenLock.sol#L3](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L3)

```solidity
pragma solidity ^0.8.7;
```

- [VRFv2RNSource.sol#L3](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L3)

#### Recommended Mitigation Steps

[Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/): Lock pragmas to specific compiler version. 

## [NC-10] There is no need to cast a variable that is already an uint, such as `uint(x)`

#### Description

There is no need to cast a variable that is already an uint, such as uint(0), 0 is also uint.

```solidity
            return (ticketSize == selectionSize) && (ticket == uint256(0));
```

#### Lines of code 

```solidity
        if (lotterySetupParams.ticketPrice == uint256(0)) {
```

- [LotterySetup.sol#L45](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L45)

```solidity
            return (ticketSize == selectionSize) && (ticket == uint256(0));
```

- [TicketUtils.sol#L32](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L32)

```solidity
            assert((winTier <= selectionSize) && (intersection == uint256(0)));
```

- [TicketUtils.sol#L99](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99)

#### Recommended Mitigation Steps

Use directly variable:

```solidity
            return (ticketSize == selectionSize) && (ticket == 0);
```

## [NC-11] Consider using `delete` rather than assigning zero to clear values    

#### Description

The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic.

#### Lines of code 

```solidity
            frontendDueTicketSales[beneficiary] = 0;
```

- [Lottery.sol#L255](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L255)

```solidity
            unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
```

- [ReferralSystem.sol#L142](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L142)

```solidity
            unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
```

- [ReferralSystem.sol#L148](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L148)

```solidity
            rewards[msg.sender] = 0;
```

- [Staking.sol#L95](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L95)

#### Recommended Mitigation Steps

Use the `delete` keyword.

## [NC-12] For functions, follow Solidity standard naming conventions

#### Description

The protocol don't follow solidity standard naming convention.

> Ref: https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-wenwin/tree/main/src)

#### Recommended Mitigation Steps

Follow solidity standard [naming convention](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions).

## [NC-13] Add NatSpec Mapping comment  

#### Description

Add NatSpec comments describing mapping keys and values.

```solidity
    mapping(address => uint256) public override rewards;
```

#### Lines of code 

```solidity
    mapping(address => uint256) private frontendDueTicketSales;
```

- [Lottery.sol#L26](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L26)

```solidity
    mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
```

- [Lottery.sol#L27](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L27)

```solidity
    mapping(uint128 => uint120) public override winningTicket;
```

- [Lottery.sol#L36](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L36)

```solidity
    mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
```

- [Lottery.sol#L37](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L37)

```solidity
    mapping(uint128 => uint256) public override ticketsSold;
```

- [Lottery.sol#L39](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L39)

```solidity
    mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets;
```

- [ReferralSystem.sol#L17](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L17)

```solidity
    mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;
```

- [ReferralSystem.sol#L19](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L19)

```solidity
    mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;
```

- [ReferralSystem.sol#L21](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L21)

```solidity
    mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;
```

- [ReferralSystem.sol#L23](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L23)

```solidity
    mapping(uint128 => uint256) public override minimumEligibleReferrals;
```

- [ReferralSystem.sol#L25](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L25)

```solidity
    mapping(address => uint256) public override userRewardPerTokenPaid;
```

- [Staking.sol#L19](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L19)

```solidity
    mapping(address => uint256) public override rewards;
```

- [Staking.sol#L20](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L20)

```solidity
    mapping(uint256 => ITicket.TicketInfo) public override ticketsInfo;
```

- [Ticket.sol#L14](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L14)

#### Recommended Mitigation Steps

```solidity
/// @dev address(user) => uint256(rewards)
    mapping(address => uint256) public override rewards;
```

## [NC-14] Constants should be defined rather than using magic numbers 

#### Description

Constants should be defined rather than using magic numbers.

#### Lines of code 

```solidity
            lotterySetupParams.selectionSize > 16 || lotterySetupParams.selectionSize >= lotterySetupParams.selectionMax
```

- [LotterySetup.sol#L61](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L61)

```solidity
            uint256 mask = uint256(type(uint16).max) << (winTier * 16);
```

- [LotterySetup.sol#L126](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L126)

```solidity
            uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
```

- [LotterySetup.sol#L127](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L127)

```solidity
            packed |= uint256(reward) << (winTier * 16);
```

- [LotterySetup.sol#L174](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L174)

#### Recommended Mitigation Steps

Define a constant.

## [NC-15] Not using the named return variables anywhere in the function is confusing

#### Description

Not using the named return variables anywhere in the function is confusing.

#### Lines of code 

```solidity
    function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
```

- [Lottery.sol#L234](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L234)

```solidity
    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
```

- [LotterySetup.sol#L120](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L120)

```solidity
    function drawScheduledAt(uint128 drawId) public view override returns (uint256 time) {
```

- [LotterySetup.sol#L152](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L152)

```solidity
    function ticketRegistrationDeadline(uint128 drawId) public view override returns (uint256 time) {
```

- [LotterySetup.sol#L156](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L156)

```solidity
    function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
```

- [PercentageMath.sol#L17](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L17)

```solidity
    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
```

- [PercentageMath.sol#L22](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/PercentageMath.sol#L22)

```solidity
        returns (uint256 minimumEligible)
```

- [ReferralSystem.sol#L115](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L115)

```solidity
    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

- [ReferralSystem.sol#L156](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L156)

```solidity
    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
```

- [ReferralSystem.sol#L161](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L161)

```solidity
    function rewardPerToken() public view override returns (uint256 _rewardPerToken) {
```

- [Staking.sol#L48](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L48)

```solidity
    function earned(address account) public view override returns (uint256 _earned) {
```

- [Staking.sol#L61](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L61)

```solidity
    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
```

- [Ticket.sol#L23](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L23)

```solidity
        returns (bool isValid)
```

- [TicketUtils.sol#L24](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L24)

```solidity
        returns (uint120 ticket)
```

- [TicketUtils.sol#L50](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L50)

```solidity
        returns (uint8 winTier)
```

- [TicketUtils.sol#L91](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L91)

#### Recommended Mitigation Steps

Consider changing the variable to be an unnamed one.
