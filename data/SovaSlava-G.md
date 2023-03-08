1. 
In most cases, contract has good previous request to chainlink. 
We shoudnt write to failedSequentialAttempts and maxFailedAttemptsReachedAt in each call of function onRandomNumberFulfilled(). Cheaper will be at first check, if lastRequestFulfilled == false. if true -> 
``
     failedSequentialAttempts = 0;
        maxFailedAttemptsReachedAt = 0;
``

so, in most cases contract will be only read from storage and dont rewrite value.
https://github.com/wenwincom/wenwin-contracts/blob/3958a8c6837ed25e244983854d60439e96a6d095/src/RNSourceController.sol#L52-53