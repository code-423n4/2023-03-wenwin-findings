# QA report

### **Total Findings: 2**

| Issue Id | Level    | Description       |
| -------- | -------- | ----------------- |
| L-01     | Low risk | [Lack of validation for constructor params](#l-01-lack-of-validation-for-constructor-params)    |
| R-01     | Refactor | [Unnecessary casting](#r-01-unnecessary-casting) |


## [L-01] Lack of validation for constructor params
**Issue:** Constructor of `StakedTokenLock` missing validation of all three input params. Wrong value for 2 out of 3 params can leave contract in unusable state. 
- `_stakedToken` is missing non-zero check. Setting zero address will leave contract in unusable state.
- `_depositDeadline` should be checked against block.timestamp. Setting lower value than current block.timestamp will not allow any deposit in contract and thus leave contract in unusable state.
- `_lockDuration` can be 0 which will effectively unlock deposited amount in next block and defeat the purpose of lock contract.

**Recommendation:** Consider adding sanity checks around all 3 constructor params

**Code:** [StakedTokenLock.sol#L16-L22](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L16-L22)
```solidity
16:    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
17:        _transferOwnership(msg.sender);
18:        stakedToken = IStaking(_stakedToken);
19:        rewardsToken = stakedToken.rewardsToken();
20:        depositDeadline = _depositDeadline;
21:        lockDuration = _lockDuration;
22:    }
```

## [R-01] Unnecessary casting
**Issue:** There is an unnecessary casting of `0` to uint256 in constructor of `LotterySetup.sol`

**Recommendation:** Consider removing uint256 casting.

**Code:** [LotterySetup.sol#L45-L47](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L45-L47)
```solidity
45:        if (lotterySetupParams.ticketPrice == uint256(0)) {
46:            revert TicketPriceZero();
47:        }
```
