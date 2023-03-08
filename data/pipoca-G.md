## 1. Cache array length to improve gas efficiency in for loop.
The current implementation uses a for loop that loads the length of the array during each iteration, which can lead to unnecessary gas consumption. Therefore, we recommend caching the length of the array before entering the loop if the length of the array does not change inside the loop. Additionally, there is no need to assign an initial value of 0 as this adds extra gas.

Here is an example of how this optimization could be implemented:

`uint length = arr.length;
for (uint i; i < length; ++i) {
    //Operations not affecting the length of the array.
}`

By caching the length of the array outside of the loop and removing the initial value assignment, we can improve gas efficiency and reduce the cost of execution.

File: Lottery.sol | Line: 125 | for (uint256 i = 0; i < drawIds.length; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L125
File: ReferralSystem.sol | Line: 35 | for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L35
File: ReferralSystem.sol | Line: 77 | for (uint256 counter = 0; counter < drawIds.length; ++counter) {
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L77
