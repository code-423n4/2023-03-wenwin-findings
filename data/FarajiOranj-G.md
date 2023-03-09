An unnecessary uint256 return variable when calling the claimable function enhances the gas consumption, and also claimWinningTicket function will be used in a loop that can cost more gas.

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L170-L174

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L259-L269

recommendation:

remove line 260
and replace line 261 with the below code:

`
        (claimedAmount, ) = this.claimable(ticketId);
`