


# [L-01] {No nonreentrant modifier used in the  functions that use safeTransfer , safeTransferFrom}


## Proof of Concept

Usage of safeTranfer, safeTransferFrom  functions  add re-entrancy possibilities, So it's always good to use  OZ Reentrancy guard . By looking at the code base from a high level overview I found some instances where safeTransferFunctions are used  and there's no nonreentrant modifiers.

---
1. Staking.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L73
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L86
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L98
2.Lottery.sol
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L130
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L175


**Impact:**
Impact is high  in a re-entracy attack  so make sure all the functions that transfer  tokens  or Eth out of the contract are well secured 
### Recommendations

use open zeppelin re-entrancy Guard and nonreentrant modifier 


# [L-02] {No checks for address(0) in  some functions and in some  constructors}

### Proof of concept
 So  this function is internal. developer  might think to use address(0 ) check inside the scope of main function . But it's better to use address(0)  check inside of internal function 
**Impact:**
User can  loss his tokens and a  Mallicious user can burn all the nativeToken sending to address(0)  

  https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RefferalSystem.sol#L74
  
```js
function mintNativeTokens(address mintTo, uint256 amount) internal override {

        ILotteryToken(address(nativeToken)).mint(mintTo, amount);

    }
```

### Recommendations

```js
 require(msg.sender != address(0));
```
or
```js
if(msg.sender == address(0)){
revert customError();
}
```

---

2.also Here

```js
function stake(uint256 amount) external override {...}
```

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L67


### in constructors

1. https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L11

```js
 constructor(address _authorizedConsumer) {

        authorizedConsumer = _authorizedConsumer;

    }
```

2.  https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L13




# [L -03 ]  poor access  control in some functions 

1. anyone can call the `getReward `  it doesn't matter but it's better  if function is called by ``onlyOwner`` or a trusted party (giving roles like manager)

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L50


```js
  function getReward() external override {

        stakedToken.getReward();

        // No need for SafeTransfer, only trusted reward token is used.

        // slither-disable-next-line unchecked-transfer

        rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));

    }
```



### Recommendations
  
  
```js
function getReward() external override onlyOwner 
```
 
---

 2 .  Anyone can call `claimReferralReward` function with    ` drawIds  `  to mint nativeTokens  it's checking `msg.sender` in  ` claimPer Draw` function but make sure to add extra security layer  to ensure that `msg.sender`  own   `drawIds` or not 

 https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L76


---


3  .  `  requireFinishedDraw(drawId);  `isn't configured correctly 


```js
function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward) {

        requireFinishedDraw(drawId);
..

        }
```
  https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RefferalSystem.sol#L137



# [Q-01] Add token balance check before buying a ticket 


## Proof of Concept

In `` buyTickets``  function  in Lottery.sol there's no check that the  ``msg.sender``    having  a   sufficient  balanceOf rewardToken or not 
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L130

## Recommendations

Add 
```js
require( rewardToken.balanceOf(msg.sender) >=  ticketPrice * tickets.length , "Insufficient rewardtokens" );

```
or  
``` js

if (rewardToken.balanceOf(msg.sender)  <  ticketPrice * tickets.length){

revert CustomError();

}

```


```js
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

       ...

        rewardToken.safeTransferFrom(msg.sender, address(this), ticketPrice * tickets.length);

    }
```




