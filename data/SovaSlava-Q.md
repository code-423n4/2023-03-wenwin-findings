1. 
```
lotterySetupParams.selectionSize > 16
``` 
should be
```
lotterySetupParams.selectionSize != 7
```
https://github.com/wenwincom/wenwin-contracts/blob/main/src/LotterySetup.sol#L61
https://docs.wenwin.com/wenwin-lottery/the-game/tickets
> During every draw, 7 numbers are randomly drawn from 35 non-repeating integer numbers 

2. If owner will not change source(if current is broken and dont send numbers) for a long time, users should have function to get their tokens back. 

3. in requestRandomNumberFromSource() in block "catch Error" should be additional code 
``
++failedSequentialAttempts;
lastRequestTimestamp = block.timestamp + maxRequestDelay;
```
because we know that request failed, and someone should be able to call retry() immediately, after "catch execute". its keep time, and user will not wait "maxRequestDelay"
https://github.com/wenwincom/wenwin-contracts/blob/3958a8c6837ed25e244983854d60439e96a6d095/src/RNSourceController.sol#L113
4. in file:
``
 if (totalTicketsSoldPrevDraw < 10_000) 
...
if (totalTicketsSoldPrevDraw < 100_000)
...
if (totalTicketsSoldPrevDraw < 1_000_000)
``
but in docs
```
<= for each if.
```
https://docs.wenwin.com/wenwin-lottery/protocol-architecture/token/rewards/referrals#referrers-allocation  -> Individual Referrer Allocation
https://github.com/wenwincom/wenwin-contracts/blob/f028970249d002c64d89a74d1ac963bef934494a/src/ReferralSystem.sol#L117