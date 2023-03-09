## Gas Optimizations 
**Total 85 instances over 9 issues**
### [G&#x2011;01] <x> += <y> costs more gas than <x> = <x> + <y> for state variables (-= too)

 Using the addition operator instead of plus-equals saves 113 gas. Subtractions act the same way.

*There are 16 instances of this issue:*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L84

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L29

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L60

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L70

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L96

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L275

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L173

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L30

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L43

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L129

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L67

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L64

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L69

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L71

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L78

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L147

### [G‑2] Using bools for storage incurs overhead
      Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27 Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past

*affected code:*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L63

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L33

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L16

### [G‑03] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if/for-statement
require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }

*There is 13 instances of this issue:*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L65

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L68

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L169

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L125

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L172

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L211

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L35

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L77

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L28

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L57

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L65

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L68

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L95

### [G‑4] internal functions only called once can be inlined to save gas
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

*There are 4 instances of this issue:*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L160

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L134

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L161

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L119

### [G-5] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

*There are 6 instances of this issue:*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L24

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L37

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L259

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L203

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L77

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L89

### [G-6] Optimize names to save gas
public/external function names and public member variable names can be optimized to save gas. See this link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted

*There is 14 instance of this issue:*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L9

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L11

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L12

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L21

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L9

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L10

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L10

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L9

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L8

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L80

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol#L50

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L36

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L12

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotteryToken.sol#L11

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStakedTokenLock.sol#L13

## [G‑7] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

There are 27 instances of this issue:

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L47

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L51

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L55

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L19

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L54

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L56

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L73

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L86

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L97

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L98

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L121

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L126

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L128

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L140

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L161

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L168

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L261

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L286

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L119

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L123

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L127

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L162

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L47

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L63

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L110

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L111

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L29

### [G‑8] Using storage instead of memory for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L160

### [G‑9] Using calldata instead of memory for read-only arguments in external functions saves gas
When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L76

