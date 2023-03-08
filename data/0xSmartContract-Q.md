## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]|No Check if OnErc721Received is implemented| 1 |
|[L-02]|Protect your NFT from copying in fork | 1 |
|[L-03]|Insufficient coverage |  |
|[L-04]|Add to Event-Emit for critical functions |1 |
|[L-05]|Loss of precision due to rounding in ` jackpotWinners `| 1 |
|[L-06]|`claimRewards` event is missing parameters | 1 |
|[L-07]|Project Upgrade and Stop Scenario should be| 1 |
|[L-08]|Add to _blacklist function_| 1 |
|[L-09]|Pragma version^0.8.19  version too recent to be trusted |  |

Total 9 issues


### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01] |Omissions in Events|1|
| [N-02] |NatSpec comments should be increased in contracts |26|
| [N-03] |`Function writing` that does not comply with the `Solidity Style Guide`| All Contracts |
| [N-04] |Include return parameters in NatSpec comments| 17 |
| [N-05] |Repeated validation logic can be converted into a function modifier|4 |
| [N-06] |For functions, follow Solidity standard naming conventions (internal function style rule) | 21 |
| [N-07] |Floating pragma| 3 |
| [N-08] |Use descriptive names for Contracts and Libraries |All Contracts  |
| [N-09] |Use SMTChecker | |
| [N-10] |Add NatSpec Mapping comment| 14  |
| [N-11] |Lines are too long|2 |
| [N-12] |Constants on the left are better | 1 |
| [N-13] |`constants` should be defined rather than using magic numbers | 9 |
| [N-14] |Use the `delete` keyword instead of assigning a value of `0` | 8 |
| [N-15] |Not using the named return variables anywhere in the function is confusing | 1 |
| [N-16] |Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked| 1 |
| [N-17] |For modern and more readable code; update import usages | 53 |
| [N-18] |Use `require` instead of `assert` | 2 |
| [N-19] |Use a single file for all system-wide constants | 13 |
| [N-20] |Use a more recent version of Solidity | 3 |
| [N-21] |Take advantage of Custom Error's return value property | 37 |
| [N-22] |Add EIP-2981 NFT Royalty Standart Support |  |
| [N-23] |Inconsistent Solidity Versions  | All Contracts |

Total 23 issues



### [L-01] No Check if OnErc721Received is implemented

`Ticket.mint()` will mint a NFT . When minting a NFT, the function does not check if a receiving contract implements onERC721Received().
`msg.sender` can be contract address.

The intention behind this function is to check if the address receiving the NFT, if it is a contract, implements onERC721Received(). Thus, there is no check whether the receiving address supports ERC-721 tokens and position could be not transferrable in some cases.

Following shows that _mint is used instead of _safeMint

```solidity

src/Ticket.sol:
  23:     function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
  26:         _mint(to, ticketId);
```

Recommendation
Consider using _safeMint instead of _mint


### [L-02] Protect your NFT from copying in fork

Occasionally, forks can happen on different blockchains.
The project will operate on the Polygon network.Recently, a fork took place on the Ethereum network as well.

There may be forked versions of Blockchains, which could cause confusion and lead to scams as duplicated NFT assets enter the market, then it could result in duplication of non-fungible tokens (NFTs) and there's likely to be some level of confusion around which assets are 'official' or 'authentic.’

A forked Blockchains, chain will thus have duplicated deeds that point to the same tokenURI

About Replay Attack:
https://twitter.com/elerium115/status/1558471934924431363?s=20&t=RRheaYJwo-GmSnePwofgag

The most important part here is NFT's tokenURI detail. If the update I suggested below is not done, Duplicate NFTs will appear as a result, potentially leading to confusion and scams.

### Tools Used
Manual Code Review


### Recommended Mitigation Steps

```solidity
src/Ticket.sol:
5: import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

```

```diff
"@openzeppelin/contracts/token/ERC721/ERC721.sol :
  135      /// @return tokenURI The URI of the queried token (path to a JSON file)
  136:     function tokenURI(uint256 _id) public view override returns (string memory) {
+ 	 if (block.chainid != 137) revert wrongChain(); // Polygon Chain ID : 137
               }

```



### [L-03] Insufficient coverage

**Description:**
The test coverage rate of the project is ~93%. Testing all functions is best practice in terms of security criteria.
Due to its capacity, test coverage is expected to be 100%.


```js
1 result - 1 file

README.md:
  190  
  191: | File                                                  | % Lines          | % Statements     | % Branches      | % Funcs         |
  192: |-------------------------------------------------------|------------------|------------------|-----------------|-----------------|
  193: | src/Lottery.sol                                       | 94.20% (65/69)   | 95.06% (77/81)   | 88.89% (16/18)  | 80.00% (12/15)  |
  194: | src/LotteryMath.sol                                   | 14.29% (2/14)    | 15.79% (3/19)    | 0.00% (0/2)     | 20.00% (1/5)    |
  195: | src/LotterySetup.sol                                  | 71.43% (20/28)   | 65.71% (23/35)   | 68.75% (11/16)  | 50.00% (3/6)    |
  196: | src/LotteryToken.sol                                  | 100.00% (3/3)    | 100.00% (3/3)    | 100.00% (2/2)   | 100.00% (1/1)   |
  197: | src/PercentageMath.sol                                | 0.00% (0/2)      | 0.00% (0/2)      | 100.00% (0/0)   | 0.00% (0/2)     |
  198: | src/RNSourceBase.sol                                  | 86.67% (13/15)   | 87.50% (14/16)   | 75.00% (6/8)    | 100.00% (2/2)   |
  199: | src/RNSourceController.sol                            | 94.74% (36/38)   | 95.00% (38/40)   | 94.44% (17/18)  | 100.00% (6/6)   |
  200: | src/ReferralSystem.sol                                | 39.53% (17/43)   | 41.67% (20/48)   | 25.00% (6/24)   | 71.43% (5/7)    |
  201: | src/Ticket.sol                                        | 100.00% (4/4)    | 100.00% (4/4)    | 100.00% (0/0)   | 100.00% (2/2)   |
  202: | src/TicketUtils.sol                                   | 100.00% (24/24)  | 100.00% (37/37)  | 75.00% (3/4)    | 100.00% (3/3)   |
  203: | src/VRFv2RNSource.sol                                 | 100.00% (4/4)    | 100.00% (4/4)    | 100.00% (2/2)   | 100.00% (2/2)   |
  204: | src/staking/StakedTokenLock.sol                       | 100.00% (10/10)  | 100.00% (10/10)  | 100.00% (4/4)   | 100.00% (3/3)   |
  205: | src/staking/Staking.sol                               | 100.00% (36/36)  | 100.00% (39/39)  | 91.67% (11/12)  | 87.50% (7/8)    |
```



### [L-04] Add to Event-Emit for critical functions

`buyTickets` is most important function in Project so event-emit should be added

```solidity

src/Lottery.sol:
  109  
  110:     function buyTickets(
  111:         uint128[] calldata drawIds,
  112:         uint120[] calldata tickets,
  113:         address frontend,
  114:         address referrer
  115:     )
  116:         external
  117:         override
  118:         requireJackpotInitialized
  119:         returns (uint256[] memory ticketIds)
  120:     {
  121:         if (drawIds.length != tickets.length) {
  122:             revert DrawsAndTicketsLenMismatch(drawIds.length, tickets.length);
  123:         }
  124:         ticketIds = new uint256[](tickets.length);
  125:         for (uint256 i = 0; i < drawIds.length; ++i) {
  126:             ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
  127:         }
  128:         referralRegisterTickets(currentDraw, referrer, msg.sender, tickets.length);
  129:         frontendDueTicketSales[frontend] += tickets.length;
  130:         rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);
  131:     }

```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit



### [L-05] Loss of precision due to rounding in ` jackpotWinners `

If `jackpotWinners` is too low then the rounding problem is big 

Add scalar so roundings are negligible

```solidity
src/Lottery.sol:

  208          if (jackpotWinners > 0) {
  209:             winAmount[drawFinalized][selectionSize] = drawRewardSize(drawFinalized, selectionSize) / jackpotWinners;
  210          } else {

```



### [L-06] `claimRewards` event is missing parameters


The `Lotterty.sol` contract has very important function; ` claimRewards ` 


However, only beneficiary, claimedAmount, rewardType are published in emits, whereas msg.sender must be specified in every transaction, a contract or web page listening to events cannot react to users, emit does not serve the purpose.
Basically, this event cannot be used


```diff
 function claimRewards(LotteryRewardType rewardType) external override returns (uint256 claimedAmount) {
        address beneficiary = (rewardType == LotteryRewardType.FRONTEND) ? msg.sender : stakingRewardRecipient;
        claimedAmount = LotteryMath.calculateRewards(ticketPrice, dueTicketsSoldAndReset(beneficiary), rewardType);

emit ClaimedRewards(beneficiary, claimedAmount, rewardType);
+    emit ClaimedRewards(msg.sender, beneficiary, claimedAmount, rewardType);

        rewardToken.safeTransfer(beneficiary, claimedAmount);
    }



```



### Recommended Mitigation Steps
add `msg.sender` parameter in event-emit


### [L-07] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol


### [L-08] Add to _blacklist function_

**Description:**
Cryptocurrency mixing service, Tornado Cash, has been blacklisted in the OFAC.
A lot of blockchain companies, token projects, NFT Projects have ```blacklisted``` all Ethereum addresses owned by Tornado Cash listed in the US Treasury Department's sanction against the protocol.
https://home.treasury.gov/policy-issues/financial-sanctions/recent-actions/20220808
In addition, these platforms even ban accounts that have received ETH on their account with Tornadocash.

Some of these Projects;
 USDC (https://www.circle.com/en/usdc)
 Flashbots (https://www.paradigm.xyz/portfolio/flashbots )
 Aave (https://aave.com/)
 Uniswap
 Balancer
 Infura
 Alchemy 
 Opensea
 dYdX 

Details on the subject;
https://twitter.com/bantg/status/1556712790894706688?s=20&t=HUTDTeLikUr6Dv9JdMF7AA


```The ban on Tornado Cash makes little sense, because in the end, no one can prevent people from using other mixer smart contracts, or forking the existing ones. It neither hinders cybercrime, nor privacy.```

Here is the most beautiful and close to the project example; Manifold

Manifold Contract
https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```js
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```
Recommended Mitigation Steps add to Blacklist function and modifier.




### [L-09] Pragma version^0.8.19  version too recent to be trusted.

https://github.com/ethereum/solidity/blob/develop/Changelog.md
0.8.19 (2023-02-22)
0.8.17 (2022-09-08)
0.8.16 (2022-08-08)
0.8.15 (2022-06-15)
0.8.10 (2021-11-09)


Unexpected bugs can be reported in recent versions;
Risks related to recent releases
Risks of complex code generation changes
Risks of new language features
Risks of known bugs

Use a non-legacy and more battle-tested version
Use 0.8.10



### [N-01] Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

```diff

src/RNSourceController.sol:
   88  
   89:     function swapSource(IRNSource newSource) external override onlyOwner {
   90:         if (address(newSource) == address(0)) {
   91:             revert RNSourceZeroAddress();
   92:         }
   93:         bool notEnoughRetryInvocations = failedSequentialAttempts < maxFailedAttempts;
   94:         bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;
   95:         if (notEnoughRetryInvocations || notEnoughTimeReachingMaxFailedAttempts) {
   96:             revert NotEnoughFailedAttempts();
   97:         }
   98:         source = newSource;
   99:         failedSequentialAttempts = 0;
  100:         maxFailedAttemptsReachedAt = 0;
  101: 
- 102:         emit SourceSet(newSource);
+ 102:         emit SourceSet(oldSource, newSource);
  103:         requestRandomNumberFromSource();
  104:     }
  81:     }

```



### [N-02] NatSpec comments should be increased in contracts

**Context:**
All project have 79 functions (without interfaces) but they have only 53 NatSpec’s

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
NatSpec comments should be increased in contracts




### [N-03] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last






### [N-04] Include return parameters in NatSpec comments

**Context:**
All project have 35 returns variable (without interfaces) but they have only 18 @return NatSpec’s


**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
Include return parameters in NatSpec comments

_Recommendation  Code Style: (from Uniswap3)_
```js
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



### [N-05] Repeated validation logic can be converted into a function modifier

If a query or logic is repeated over many lines, using a modifier improves the readability and reusability of the code
!= address(0))

```solidity
4 results - 3 files

src/ReferralSystem.sol:
  59      {
  60:         if (referrer != address(0)) {
  61              uint256 minimumEligible = minimumEligibleReferrals[currentDraw];

src/RNSourceController.sol:
  80          }
  81:         if (address(source) != address(0)) {
  82              revert AlreadyInitialized();

src/staking/Staking.sol:
  108      function _beforeTokenTransfer(address from, address to, uint256) internal override {
  109:         if (from != address(0)) {
  110              _updateReward(from);

  112  
  113:         if (to != address(0)) {
  114              _updateReward(to);
```




### [N-06] For functions, follow Solidity standard naming conventions (internal function style rule)

```solidity


21 results - 8 files

src/Lottery.sol:
  203:     function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw {
  279:     function requireFinishedDraw(uint128 drawId) internal view override {
  285:     function mintNativeTokens(address mintTo, uint256 amount) internal override {

src/LotteryMath.sol:
  62:     function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot) {

src/PercentageMath.sol:
  17:     function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
  22:     function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {

src/ReferralSystem.sol:
   74:     function mintNativeTokens(address mintTo, uint256 amount) internal virtual;
   87:     function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {
  134:     function requireFinishedDraw(uint128 drawId) internal view virtual;
  156:     function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
  161:     function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {

src/RNSourceBase.sol:
   8:     address internal immutable authorizedConsumer;
   9:     mapping(uint256 => RandomnessRequest) internal requests;
  33:     function fulfill(uint256 requestId, uint256 randomNumber) internal {
  48:     function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId);

src/RNSourceController.sol:
  38:     function requestRandomNumber() internal {
  58:     function receiveRandomNumber(uint256 randomNumber) internal virtual;

src/Ticket.sol:
  19:     function markAsClaimed(uint256 ticketId) internal {
  23:     function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {

src/VRFv2RNSource.sol:
  28:     function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId) {
  32:     function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override {

```


**Description:**
The above codes don't follow Solidity's standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)



### [N-07] Floating pragma

**Description:**
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List: 

```solidity
3 results - 3 files

src/VRFv2RNSource.sol:
  2  // slither-disable-next-line solc-version
  3: pragma solidity ^0.8.7;
  4  

src/interfaces/IVRFv2RNSource.sol:
  2  // slither-disable-next-line solc-version
  3: pragma solidity ^0.8.7;
  4  

src/staking/StakedTokenLock.sol:
  2  // slither-disable-next-line solc-version
  3: pragma solidity ^0.8.17;

```


**Recommendation:**
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.



### [N-08] Use descriptive names for Contracts and Libraries


This codebase will be difficult to navigate, as there are no descriptive naming conventions that specify which files should contain meaningful logic.

Prefixes should be added like this by filing:

Interface I_
absctract contracts Abs_
Libraries Lib_

We recommend that you implement this or a similar agreement.


### [N-09] Use SMTChecker

The *highest* tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19


### [N-10]  Add NatSpec Mapping comment

**Description:**
Add NatSpec comments describing mapping keys and values


**Context:**

```solidity
14 results - 5 files

src/Lottery.sol:
  26:     mapping(address => uint256) private frontendDueTicketSales;
  27:     mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
  36:     mapping(uint128 => uint120) public override winningTicket;
  37:     mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
  39:     mapping(uint128 => uint256) public override ticketsSold;

src/ReferralSystem.sol:
  17:     mapping(uint128 => mapping(address => UnclaimedTicketsData)) public override unclaimedTickets; 
  19:     mapping(uint128 => uint256) public override totalTicketsForReferrersPerDraw;
  21:     mapping(uint128 => uint256) public override referrerRewardPerDrawForOneTicket;
  23:     mapping(uint128 => uint256) public override playerRewardsPerDrawForOneTicket;
  25:     mapping(uint128 => uint256) public override minimumEligibleReferrals;
  26  

src/RNSourceBase.sol:
  9:     mapping(uint256 => RandomnessRequest) internal requests;

src/Ticket.sol:
  14:     mapping(uint256 => ITicket.TicketInfo) public override ticketsInfo;
  15  

src/staking/Staking.sol:
  19:     mapping(address => uint256) public override userRewardPerTokenPaid;
  20:     mapping(address => uint256) public override rewards;

```

**Recommendation:**

```solidity

 /// @dev    address(user) -> address(ERC0 Contract Address) -> uint256(allowance amount from user)
    mapping(address => mapping(address => uint256)) public allowance;

```


### [N-11] Lines are too long

[LotteryMath.sol#L62](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L62)

[IStaking.sol#L35](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol#L35)


**Description:**
It is generally recommended that lines in the source code should not exceed 80-120 characters. Today's screens are much larger, so in some cases it makes sense to expand that.The lines above should be split when they reach that length, as the files will most likely be on GitHub and GitHub always uses a scrollbar when the length is more than 164 characters.

(why-is-80-characters-the-standard-limit-for-code-width)[https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width]

**Recommendation:**
Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction

```js
thisFunctionCallIsReallyLong(
    longArgument1,
    longArgument2,
    longArgument3
);
```

### [N-12] Constants on the left are better

If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns). 

https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

```solidity

10 results - 4 files

src/Lottery.sol:
  261          (claimedAmount, winTier) = this.claimable(ticketId);
  262:         if (claimedAmount == 0) {
  263              revert NothingToClaim(ticketId);

src/LotterySetup.sol:
   47          }
   48:         if (lotterySetupParams.selectionSize == 0) {
   49              revert SelectionSizeZero();

  105          // slither-disable-next-line incorrect-equality
  106:         if (initialPot == 0) {
  107              revert JackpotNotInitialized();

  122              return _baseJackpot(initialPot);
  123:         } else if (winTier == 0 || winTier > selectionSize) {
  124              return 0;

src/ReferralSystem.sol:
  31      ) {
  32:         if (_rewardsToReferrersPerDraw.length == 0) {
  33              revert ReferrerRewardsInvalid();

  35          for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
  36:             if (_rewardsToReferrersPerDraw[i] == 0) {
  37                  revert ReferrerRewardsInvalid();

  88          // if no tickets sold there is no incentives, so no rewards to be set
  89:         if (ticketsSoldDuringDraw == 0) {
  90              return;

src/staking/Staking.sol:
  49          uint256 _totalSupply = totalSupply();
  50:         if (_totalSupply == 0) {
  51              return rewardPerTokenStored;

  68          // _updateReward is not needed here as it's handled by _beforeTokenTransfer
  69:         if (amount == 0) {
  70              revert ZeroAmountInput();

  80          // _updateReward is not needed here as it's handled by _beforeTokenTransfer
  81:         if (amount == 0) {
  82              revert ZeroAmountInput();
```


### [N-13] `constants` should be defined rather than using magic numbers

A magic number is a numeric literal that is used in the code without any explanation of its meaning. The use of magic numbers makes programs less readable and hence more difficult to maintain and update.
Even assembly can benefit from using readable constants instead of hex/numeric literals

```solidity

9 results - 5 files

src/LotteryMath.sol:
  14:     uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;
  16:     uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;
  20:     uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;
  22:     uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;

src/LotterySetup.sol:
  36:     uint256 private constant BASE_JACKPOT_PERCENTAGE = 30_030; // 30.03%

src/LotteryToken.sol:
  12:     uint256 public constant override INITIAL_SUPPLY = 1_000_000_000e18;

src/PercentageMath.sol:
   8:     uint256 public constant PERCENTAGE_BASE = 100_000;
  11:     uint256 public constant ONE_PERCENT = 1000;

src/RNSourceController.sol:
  19:     uint256 private constant MAX_MAX_FAILED_ATTEMPTS = 10;
```




### [N-14] Use the `delete` keyword instead of assigning a value of `0`


Using the 'delete' keyword instead of assigning a '0' value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use `delete` instead `0` value assign , it will be gas saved

```solidity
8 results - 4 files

src/Lottery.sol:
  255:             frontendDueTicketSales[beneficiary] = 0;

src/ReferralSystem.sol:
  142:             unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
  148:             unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;

src/RNSourceController.sol:
   52:         failedSequentialAttempts = 0;
   53:         maxFailedAttemptsReachedAt = 0;
   99:         failedSequentialAttempts = 0;
  100:         maxFailedAttemptsReachedAt = 0;

src/staking/Staking.sol:
  95:             rewards[msg.sender] = 0;
```


### [N-15] Not using the named return variables anywhere in the function is confusing


```solidity

1 result - 1 file

src/Lottery.sol:
  233  
  234:     function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
  235:         return drawRewardSize(currentDraw, winTier);
  236:     }
```

**Recommendation:**
Consider adopting a consistent approach to return values throughout the codebase by removing all named return variables, explicitly declaring them as local variables, and adding the necessary return statements where appropriate. This would improve both the explicitness and readability of the code, and it may also help reduce regressions during future code refactors.



### [N-16] Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked


```diff


1 result - 1 file

src/Lottery.sol:
  169  
  170:     function claimWinningTickets(uint256[] calldata ticketIds) external override returns (uint256 claimedAmount) {
+                require(ticketIds.length< max ticketIdsLengt, "max length");
  171:         uint256 totalTickets = ticketIds.length;
  172:         for (uint256 i = 0; i < totalTickets; ++i) {
  173:             claimedAmount += claimWinningTicket(ticketIds[i]);
  174:         }
  175:         rewardToken.safeTransfer(msg.sender, claimedAmount);
  176:     }
```

**Recommendation:**
Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to gas limitations or failed transactions. Consider bounding the loop length if the array is expected to be growing and/or handling a huge list of elements to avoid unnecessary gas wastage and denial of service.

### [N-17] For modern and more readable code; update import usages

**Context:**
```solidity

53 results - 20 files

src/Lottery.sol:
   4  
   5: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
   6: import "@openzeppelin/contracts/utils/math/Math.sol";
   7: import "src/ReferralSystem.sol";
   8: import "src/RNSourceController.sol";
   9: import "src/staking/Staking.sol";
  10: import "src/LotterySetup.sol";
  11: import "src/TicketUtils.sol";
  12  

src/LotteryMath.sol:
  4  
  5: import "src/interfaces/ILottery.sol";
  6: import "src/PercentageMath.sol";
  7  

src/LotterySetup.sol:
   4  
   5: import "@openzeppelin/contracts/utils/math/Math.sol";
   6: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
   7: import "src/PercentageMath.sol";
   8: import "src/LotteryToken.sol";
   9: import "src/interfaces/ILotterySetup.sol";
  10: import "src/Ticket.sol";
  11  

src/LotteryToken.sol:
  4  
  5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
  6: import "src/interfaces/ILotteryToken.sol";
  7: import "src/LotteryMath.sol";
  8  

src/ReferralSystem.sol:
  4  
  5: import "@openzeppelin/contracts/utils/math/Math.sol";
  6: import "src/interfaces/IReferralSystem.sol";
  7: import "src/PercentageMath.sol";
  8  

src/RNSourceBase.sol:
  4  
  5: import "src/interfaces/IRNSource.sol";
  6  

src/RNSourceController.sol:
  4  
  5: import "@openzeppelin/contracts/access/Ownable2Step.sol";
  6: import "src/interfaces/IRNSource.sol";
  7: import "src/interfaces/IRNSourceController.sol";
  8  

src/Ticket.sol:
  4  
  5: import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
  6: import "src/interfaces/ITicket.sol";
  7  

src/VRFv2RNSource.sol:
  4  
  5: import "@chainlink/contracts/src/v0.8/VRFV2WrapperConsumerBase.sol";
  6: import "src/interfaces/IVRFv2RNSource.sol";
  7: import "src/RNSourceBase.sol";
  8  

src/interfaces/ILottery.sol:
  4  
  5: import "src/interfaces/ILotterySetup.sol";
  6: import "src/interfaces/IRNSourceController.sol";
  7: import "src/interfaces/ITicket.sol";
  8: import "src/interfaces/IReferralSystem.sol";
  9  

src/interfaces/ILotterySetup.sol:
  4  
  5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
  6: import "src/interfaces/ITicket.sol";
  7  

src/interfaces/ILotteryToken.sol:
  4  
  5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
  6  

src/interfaces/IReferralSystem.sol:
  4  
  5: import "src/interfaces/ILotteryToken.sol";
  6  

src/interfaces/IRNSourceController.sol:
  4  
  5: import "@openzeppelin/contracts/access/Ownable2Step.sol";
  6: import "src/interfaces/IRNSource.sol";
  7  

src/interfaces/ITicket.sol:
  4  
  5: import "@openzeppelin/contracts/token/ERC721/IERC721.sol";
  6  

src/interfaces/IVRFv2RNSource.sol:
  4  
  5: import "src/interfaces/IRNSource.sol";
  6  

src/staking/StakedTokenLock.sol:
  4  
  5: import "@openzeppelin/contracts/access/Ownable2Step.sol";
  6: import "src/staking/interfaces/IStakedTokenLock.sol";
  7: import "src/staking/interfaces/IStaking.sol";
  8  

src/staking/Staking.sol:
  4  
  5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
  6: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
  7: import "src/interfaces/ILottery.sol";
  8: import "src/LotteryMath.sol";
  9: import "src/staking/interfaces/IStaking.sol";
  10  

src/staking/interfaces/IStakedTokenLock.sol:
  4  
  5: import "src/staking/interfaces/IStaking.sol";
  6  

src/staking/interfaces/IStaking.sol:
  4  
  5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
  6: import "src/interfaces/ILottery.sol";


```
**Description:**
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

**Recommendation:**
`import {contract1 , contract2} from "filename.sol";`

A good example from the ArtGobblers project;
```js
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```


### [N-18] Use `require` instead of `assert`

```solidity

2 results - 2 files

src/LotterySetup.sol:
  146          // must hold after this call, this will be used as a check that jackpot is initialized
  147:         assert(initialPot > 0);
  148  

src/TicketUtils.sol:
  98              }
  99:             assert((winTier <= selectionSize) && (intersection == uint256(0)));
  100          }

```

**Description:**
Assert should not be used except for tests, `require` should be used

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process's available gas instead of returning it, as request()/revert() did.

assert() and ruqire();
The big difference between the two is that the `assert()`function when false, uses up all the remaining gas and reverts all the changes made.
Meanwhile, a  `require()` function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay.
This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states "The Assert function generates an error of type Panic(uint256). Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there's a mistake".

### [N-19] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values)

  This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

```solidity
13 results - 6 files

src/LotteryMath.sol:
  14:     uint256 public constant STAKING_REWARD = 20 * PercentageMath.ONE_PERCENT;
  16:     uint256 public constant FRONTEND_REWARD = 10 * PercentageMath.ONE_PERCENT;
  18:     uint256 public constant TICKET_PRICE_TO_POT = PercentageMath.PERCENTAGE_BASE - STAKING_REWARD - FRONTEND_REWARD;
  20:     uint256 public constant SAFETY_MARGIN = 67 * PercentageMath.ONE_PERCENT;
  22:     uint256 public constant EXCESS_BONUS_ALLOCATION = 50 * PercentageMath.ONE_PERCENT;
  24:     uint128 public constant DRAWS_PER_YEAR = 52;
  25  

src/LotterySetup.sol:
  36:     uint256 private constant BASE_JACKPOT_PERCENTAGE = 30_030; // 30.03% 

src/LotteryToken.sol:
  12:     uint256 public constant override INITIAL_SUPPLY = 1_000_000_000e18;

src/PercentageMath.sol:
   8:     uint256 public constant PERCENTAGE_BASE = 100_000;
  11:     uint256 public constant ONE_PERCENT = 1000;

src/RNSourceController.sol:
  18      uint256 public immutable override maxRequestDelay;
  19:     uint256 private constant MAX_MAX_FAILED_ATTEMPTS = 10;
  20:     uint256 private constant MAX_REQUEST_DELAY = 5 hours;

```

### [N-20] Use a more recent version of Solidity

**Context:**

```solidity
3 results - 3 files

src/VRFv2RNSource.sol:
  3: pragma solidity ^0.8.7;

src/interfaces/IVRFv2RNSource.sol:
  3: pragma solidity ^0.8.7;

src/staking/StakedTokenLock.sol:
  3: pragma solidity ^0.8.17;
```

**Recommendation:**
Old version of Solidity is used ,newer version can be used `(0.8.19)` 


### [N-21] Take advantage of Custom Error's return value property

An important feature of Custom Error is that values such as address, tokenID, msg.value can 
be written inside the `()` sign, this kind of approach provides a serious advantage in 
debugging and examining the revert details of dapps such as tenderly.


```solidity

37 results - 7 files

src/Lottery.sol:
   46:             revert InvalidTicket();
   54:             revert DrawAlreadyInProgress();
   62:             revert DrawNotInProgress();
  136:             revert ExecutingDrawTooEarly();

src/LotterySetup.sol:
   43:             revert RewardTokenZero();
   46:             revert TicketPriceZero();
   49:             revert SelectionSizeZero();
   52:             revert SelectionSizeMaxTooBig();
   58:             revert InvalidExpectedPayout();
   63:             revert SelectionSizeTooBig();
   69:             revert DrawPeriodInvalidSetup();
   75:             revert InitialPotPeriodTooShort();
  107:             revert JackpotNotInitialized();
  134:             revert JackpotAlreadyInitialized();
  138:             revert FinalizingInitialPotBeforeDeadline();
  166:             revert InvalidFixedRewardSetup();
  172:                 revert InvalidFixedRewardSetup();

src/LotteryToken.sol:
  24:             revert UnauthorizedMint();

src/ReferralSystem.sol:
  33:             revert ReferrerRewardsInvalid();
  37:                 revert ReferrerRewardsInvalid();

src/RNSourceController.sol:
  28:             revert MaxFailedAttemptsTooBig();
  31:             revert MaxRequestDelayTooBig();
  40:             revert PreviousRequestNotFulfilled();
  48:             revert RandomNumberFulfillmentUnauthorized();
  62:             revert CannotRetrySuccessfulRequest();
  65:             revert CurrentRequestStillActive();
  79:             revert RNSourceZeroAddress();
  82:             revert AlreadyInitialized();
  91:             revert RNSourceZeroAddress();
  96:             revert NotEnoughFailedAttempts();

src/staking/StakedTokenLock.sol:
  27:             revert DepositPeriodOver();
  40:             revert LockPeriodOngoing();

src/staking/Staking.sol:
  32:             revert ZeroAddressInput();
  35:             revert ZeroAddressInput();
  38:             revert ZeroAddressInput();
  70:             revert ZeroAmountInput();
  82:             revert ZeroAmountInput();

```

### [N-22] Add EIP-2981 NFT Royalty Standart Support

Consider adding EIP-2981 NFT Royalty Standard to the project

https://eips.ethereum.org/EIPS/eip-2981


Royalty (Copyright – EIP 2981):

Fixed % royalties: For example, 6% of all sales go back to artists
Declining royalties: There may be a continuous decline in sales based on time or any other variable.
Dynamic royalties : Varies over time or sales amount
Upgradeable royalties: Allows a legal entity or NFT owner to change any copyright
Incremental royalties: No royalties, for example when sold for less than $100
Managed royalties : Funds are owned by a DAO, imagine the recipient is a DAO treasury
Royalties to different people : Collectors and artists can even use royalties, not specific to a particular personality


### [N-23 ] Inconsistent Solidity Versions 

**Description:**
Different Solidity compiler versions are used throughout Src repositories. The following contracts mix versions: 

```js
pragma solidity 0.8.19
pragma solidity ^0.8.7
pragma solidity ^0.8.17
```

**Recommendation:**
Versions must be consistent with each other.

