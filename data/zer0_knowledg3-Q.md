| Number | Issues details                             |
| ------ | -------------------------------------------|
| L-01   | no emitting of event for important function|

# [L-01] no emitting of event for important function

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L170
function need to have an event.

### Recommended Steps to save gas
emit an event for that function with tokenId and claimedAmount




