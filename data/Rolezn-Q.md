## Summary<a name="Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Add to `blacklist` function | 4 |
| [LOW&#x2011;2](#LOW&#x2011;2) | `decimals()` not part of ERC20 standard | 3 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Event is missing parameters | 1 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Possible rounding issue | 2 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Use `_safeMint` instead of `_mint` | 1 |
| [LOW&#x2011;6](#LOW&#x2011;6) | Contracts are not using their OZ Upgradeable counterparts | 16 |
| [LOW&#x2011;7](#LOW&#x2011;7) | Remove unused code | 1 |
| [LOW&#x2011;8](#LOW&#x2011;8) | `require()` should be used instead of `assert()` | 2 |

Total: 30 contexts over 8 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Avoid Floating Pragmas: The Version Should Be Locked | 3 |
| [NC&#x2011;2](#NC&#x2011;2) | Constants Should Be Defined Rather Than Using Magic Numbers | 3 |
| [NC&#x2011;3](#NC&#x2011;3) | Constants in comparisons should appear on the left side | 1 |
| [NC&#x2011;4](#NC&#x2011;4) | Function writing that does not comply with the Solidity Style Guide  | 24 |
| [NC&#x2011;5](#NC&#x2011;5) | Use `delete` to Clear Variables | 8 |
| [NC&#x2011;6](#NC&#x2011;6) | NatSpec return parameters should be included in contracts | 1 |
| [NC&#x2011;7](#NC&#x2011;7) | Implementation contract may not be initialized | 10 |
| [NC&#x2011;8](#NC&#x2011;8) | NatSpec comments should be increased in contracts | 1 |
| [NC&#x2011;9](#NC&#x2011;9) | Use a more recent version of Solidity | 3 |
| [NC&#x2011;10](#NC&#x2011;10) | Public Functions Not Called By The Contract Should Be Declared External Instead | 1 |
| [NC&#x2011;11](#NC&#x2011;11) | Empty blocks should be removed or emit something | 1 |

Total: 56 contexts over 11 issues

## Low Risk Issues

### <a href="#Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Add to `blacklist` function

NFT thefts have increased recently, so with the addition of hacked NFT lottery tickets to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```


#### <ins>Recommended Mitigation Steps</ins>
Add to Blacklist function and modifier.



### <a href="#Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> `decimals()` not part of ERC20 standard

`decimals()` is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

#### <ins>Proof Of Concept</ins>


```solidity
79: uint256 tokenUnit = 10 ** IERC20Metadata(address(lotterySetupParams.token)).decimals()
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L79

```solidity
128: return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals()
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L128

```solidity
168: uint256 divisor = 10 ** (IERC20Metadata(address(rewardToken)).decimals()
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L168






### <a href="#Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Event is missing parameters

The following functions are missing critical parameters when emitting an event.
When dealing with source address which uses the value of `msg.sender`, the `msg.sender` value must be specified in every transaction, a contract or web page listening to events cannot react to users, emit does not serve the purpose. Basically, this event cannot be used.

#### <ins>Proof Of Concept</ins>


```solidity

function claimRewards(LotteryRewardType rewardType) external override returns (uint256 claimedAmount) {
        address beneficiary = (rewardType == LotteryRewardType.FRONTEND) ? msg.sender : stakingRewardRecipient;
        claimedAmount = LotteryMath.calculateRewards(ticketPrice, dueTicketsSoldAndReset(beneficiary), rewardType);

        emit ClaimedRewards(beneficiary, claimedAmount, rewardType);
        rewardToken.safeTransfer(beneficiary, claimedAmount);
    }

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L155




#### <ins>Recommended Mitigation Steps</ins>
Add `msg.sender` parameter in event-emit





### <a href="#Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Possible rounding issue

Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator

#### <ins>Proof Of Concept</ins>


```solidity
100: referrerRewardForDraw / totalTicketsForReferrersPerCurrentDraw;
105: playerRewardsPerDrawForOneTicket[drawFinalized] = playerRewardForDraw / ticketsSoldDuringDraw;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L100

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L105









### <a href="#Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Use `_safeMint` instead of `_mint`

According to openzepplin's ERC721, the use of `_mint` is discouraged, use _safeMint whenever possible.
https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-_mint-address-uint256-

#### <ins>Proof Of Concept</ins>


```solidity
26: _mint(to, ticketId);
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol#L26




#### <ins>Recommended Mitigation Steps</ins>

Use `_safeMint` whenever possible instead of `_mint`



### <a href="#Summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

```solidity
5: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
6: import "@openzeppelin/contracts/utils/math/Math.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L5-L6

```solidity
5: import "@openzeppelin/contracts/utils/math/Math.sol";
6: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L5-L6

```solidity
5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryToken.sol#L5

```solidity
5: import "@openzeppelin/contracts/utils/math/Math.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L5

```solidity
5: import "@openzeppelin/contracts/access/Ownable2Step.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L5

```solidity
5: import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol#L5

```solidity
5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotterySetup.sol#L5

```solidity
5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ILotteryToken.sol#L5

```solidity
5: import "@openzeppelin/contracts/access/Ownable2Step.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IRNSourceController.sol#L5

```solidity
5: import "@openzeppelin/contracts/token/ERC721/IERC721.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/ITicket.sol#L5

```solidity
5: import "@openzeppelin/contracts/access/Ownable2Step.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L5

```solidity
5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
6: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L5-L6

```solidity
5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/interfaces/IStaking.sol#L5



#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.
See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for list of available upgradeable contracts




### <a href="#Summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Remove unused code
This code is not used in the main project contract files, remove it or add event-emit
Code that is not in use, suggests that they should not be present and could potentially contain insecure functionalities.

#### <ins>Proof Of Concept</ins>


```solidity
function fulfillRandomWords
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L32






### <a href="#Summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> `require()` Should Be Used Instead Of `assert()`

Prior to solidity version 0.8.0, hitting an assert consumes the remainder of the transaction's available gas rather than returning it, as `require()`/`revert()` do. `assert()` should be avoided even past solidity version 0.8.0 as its <a href="https://docs.soliditylang.org/en/v0.8.14/control-structures.html#panic-via-assert-and-error-via-require">documentation</a> states that "The assert function creates an error of type Panic(uint256). ... Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix".

#### <ins>Proof Of Concept</ins>


```solidity
147: assert(initialPot > 0);

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L147

```solidity
99: assert((winTier <= selectionSize) && (intersection == uint256(0)));

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/TicketUtils.sol#L99







### <a href="#Summary">[LOW&#x2011;12]</a><a name="LOW&#x2011;12"> Admin privilege - A single point of failure can allow a hacked or malicious owner use critical functions in the project

The `owner` role has a single point of failure and `onlyOwner` can use critical a few functions.

`owner` role in the project:
Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

#### Impact
Hacked owner or malicious owner can immediately use critical functions in the project.

#### <ins>Proof Of Concept</ins>

```solidity
77: function initSource(IRNSource rnSource) external override onlyOwner {

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L77

```solidity
89: function swapSource(IRNSource newSource) external override onlyOwner {

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L89

```solidity
24: function deposit(uint256 amount) external override onlyOwner {

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L24

```solidity
37: function withdraw(uint256 amount) external override onlyOwner {

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L37



#### <ins>Recommended Mitigation Steps</ins>
Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.
Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

#### Reference
The status quo regarding significant centralization vectors has always been to award M severity but I have lowered it to QA as it is a common finding this is also in order to at least warn users of the protocol of this category of risks. See <a href="https://gist.github.com/GalloDaSballo/881e7a45ac14481519fb88f34fdb8837">here</a> for list of centralization issues previously judged.




## Non Critical Issues

### <a href="#Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Avoid Floating Pragmas: The Version Should Be Locked

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### <ins>Proof Of Concept</ins>

```solidity
Found usage of floating pragmas ^0.8.7 of Solidity in [VRFv2RNSource.sol]
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L3

```solidity
Found usage of floating pragmas ^0.8.7 of Solidity in [IVRFv2RNSource.sol]
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IVRFv2RNSource.sol#L3

```solidity
Found usage of floating pragmas ^0.8.17 of Solidity in [StakedTokenLock.sol]
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L3






### <a href="#Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Constants Should Be Defined Rather Than Using Magic Numbers

#### <ins>Proof Of Concept</ins>


```solidity
117: if (totalTicketsSoldPrevDraw < 10_000) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L117

```solidity
121: if (totalTicketsSoldPrevDraw < 100_000) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L121

```solidity
125: if (totalTicketsSoldPrevDraw < 1_000_000) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L125






### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Constants in comparisons should appear on the left side

Doing so will prevent <a href="https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html">typo bugs</a>

#### <ins>Proof Of Concept</ins>

```solidity
50: if (_totalSupply == 0) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L50






### <a href="#Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Function writing that does not comply with the Solidity Style Guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

#### <ins>Proof Of Concept</ins>

All in-scope contracts

For example: See `Lottery.sol`




### <a href="#Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Use `delete` to Clear Variables

`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at `x`.

The `delete` key better conveys the intention and is also more idiomatic. Consider replacing assignments of zero with `delete` statements.

#### <ins>Proof Of Concept</ins>

```solidity
255: frontendDueTicketSales[beneficiary] = 0;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L255

```solidity
142: unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
148: unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L142

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L148



```solidity
52: failedSequentialAttempts = 0;
53: maxFailedAttemptsReachedAt = 0;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L52-L53

```solidity
99: failedSequentialAttempts = 0;
100: maxFailedAttemptsReachedAt = 0;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L99-L100

```solidity
95: rewards[msg.sender] = 0;

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L95







### <a href="#Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> NatSpec return parameters should be included in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
#### <ins>Proof Of Concept</ins>


All in-scope contracts

For example: See `Lottery.sol` for `function registerTicket(`

#### <ins>Recommended Mitigation Steps</ins>

Include return parameters in NatSpec comments

Recommendation Code Style: (from Uniswap3)

```solidity
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```




### <a href="#Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Implementation contract may not be initialized

OpenZeppelin recommends that the initializer modifier be applied to constructors. 
Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.
https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### <ins>Proof Of Concept</ins>


```solidity
84: constructor(
        LotterySetupParams memory lotterySetupParams,
        uint256 playerRewardFirstDraw,
        uint256 playerRewardDecreasePerDraw,
        uint256[] memory rewardsToReferrersPerDraw,
        uint256 maxRNFailedAttempts,
        uint256 maxRNRequestDelay
    )
        Ticket()
        LotterySetup(lotterySetupParams)
        ReferralSystem(playerRewardFirstDraw, playerRewardDecreasePerDraw, rewardsToReferrersPerDraw)
        RNSourceController(maxRNFailedAttempts, maxRNRequestDelay)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L84

```solidity
41: constructor(LotterySetupParams memory lotterySetupParams)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotterySetup.sol#L41

```solidity
17: constructor() ERC20("Wenwin Lottery", "LOT")
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/LotteryToken.sol#L17

```solidity
27: constructor(
        uint256 _playerRewardFirstDraw,
        uint256 _playerRewardDecreasePerDraw,
        uint256[] memory _rewardsToReferrersPerDraw
    )
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/ReferralSystem.sol#L27

```solidity
11: constructor(address _authorizedConsumer)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceBase.sol#L11

```solidity
26: constructor(uint256 _maxFailedAttempts, uint256 _maxRequestDelay)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/RNSourceController.sol#L26

```solidity
17: constructor() ERC721("Wenwin Lottery Ticket", "WLT")
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol#L17

```solidity
13: constructor(
        address _authorizedConsumer,
        address _linkAddress,
        address _wrapperAddress,
        uint16 _requestConfirmations,
        uint32 _callbackGasLimit
    )
        RNSourceBase(_authorizedConsumer)
        VRFV2WrapperConsumerBase(_linkAddress, _wrapperAddress)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L13

```solidity
16: constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L16

```solidity
22: constructor(
        ILottery _lottery,
        IERC20 _rewardsToken,
        IERC20 _stakingToken,
        string memory name,
        string memory symbol
    )
        ERC20(name, symbol)
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/Staking.sol#L22





### <a href="#Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> NatSpec comments should be increased in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

#### <ins>Proof Of Concept</ins>


All in-scope contracts

For example: See `Lottery.sol`

#### <ins>Recommended Mitigation Steps</ins>

NatSpec comments should be increased in contracts



### <a href="#Summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Use a more recent version of Solidity

<a href="https://blog.soliditylang.org/2022/02/16/solidity-0.8.12-release-announcement/">0.8.12</a>: 
string.concat() instead of abi.encodePacked(<str>,<str>)

<a href="https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/">0.8.13</a>: 
Ability to use using for with a list of free functions

<a href="https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/">0.8.14</a>:

ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.

<a href="https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/">0.8.15</a>:

Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.

<a href="https://blog.soliditylang.org/2022/08/08/solidity-0.8.16-release-announcement/">0.8.16</a>:

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

<a href="https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/">0.8.17</a>:

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

<a href="https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/">0.8.19</a>:
SMTChecker: New trusted mode that assumes that any compile-time available code is the actual used code, even in external calls. 
Bug Fixes: 
- Assembler: Avoid duplicating subassembly bytecode where possible.
- Code Generator: Avoid including references to the deployed label of referenced functions if they are called right away.
- ContractLevelChecker: Properly distinguish the case of missing base constructor arguments from having an unimplemented base function.
- SMTChecker: Fix internal error caused by unhandled z3 expressions that come from the solver when bitwise operators are used.
- SMTChecker: Fix internal error when using the custom NatSpec annotation to abstract free functions.
- TypeChecker: Also allow external library functions in using for.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/VRFv2RNSource.sol#L3

```solidity
pragma solidity ^0.8.7;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/interfaces/IVRFv2RNSource.sol#L3

```solidity
pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/staking/StakedTokenLock.sol#L3



#### <ins>Recommended Mitigation Steps</ins>

Consider updating to a more recent solidity version.



### <a href="#Summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Public Functions Not Called By The Contract Should Be Declared External Instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public.

#### <ins>Proof Of Concept</ins>


```solidity
function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Lottery.sol#L234





### <a href="#Summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> Empty blocks should be removed or emit something

#### <ins>Proof Of Concept</ins>

```solidity
17: constructor() ERC721("Wenwin Lottery Ticket", "WLT") { }

```

https://github.com/code-423n4/2023-03-wenwin/tree/main/src/Ticket.sol#L17




