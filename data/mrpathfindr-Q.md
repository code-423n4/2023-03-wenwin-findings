Potential Mis-calculation of referrerTicketCount in RefferalSystem.sol 
---

Low impact finding: 

Proof of Concept - in src/ReferralSystem.sol

The function `referralRegisterTickets()` takes in the value `uint256 numberOfTickets`
This value is then converted to `uint128` here 


`  unclaimedTickets[currentDraw][referrer].referrerTicketCount += uint128(numberOfTickets);`
and here 
`  unclaimedTickets[currentDraw][player].referrerTicketCount += uint128(numberOfTickets);`


If the parameter `uint256 numberOfTickets` is greater than the max value of `uint128` this will result in truncation, meaning that the upper bits of the uint256 value will be lost. This could result in unexpected behavior in the contract, since the actual value passed in will not be the same as the value that is used in the contract.


## Recommended Mitigation Steps

It's generally a good practice to use the data type that can handle the maximum value that the function could receive. If the function could potentially receive a `uint256`, then it would be better to use uint256 throughout the function instead of converting to `uint128`

 

Options to create nearly impossible lotteries — Unnecessary
---

Impact: Low, potentially gas saving


Considering the conditions for setting up a lottery. The following conditions produce the largest odds possible in the protocol. Using this calculator: `https://www.easysurf.cc/lotodd.htm`

selectionMax =  120
selectionSize = 16

which is 16/120 odds 

The odds of winning here are 1 in 31,044,058,215,401,408,464  combinations 

As mentioned by sponsors the most common lottery is 7/35

The odds of winning here are 1 in 6,724,520 

A significant difference between odds that can be created and realistic odds. 

Tools Used 
https://www.easysurf.cc/lotodd.htm

## Recommended Mitigation Steps

I recommend setting a maximum odds limit to avoid users creating nearly impossible Lotteries
The ratio between selectionMax & selectionSize  can be used to create this limit 


e.g if ( selectionMax / selectionSize < 0.2){
revert oddsTooLarge
}

This would drastically improve the quality of competitions, could potentially save gas well well since there would be less combinations to sift through. 

Missing dev Comments for Key functions 
---

Although there are @dev comments, this could be significantly improved to reduce the need for clarification on functions. 

Critical Functions that could use comments:

buyTickets()

executeDraw()

claimWinningTickets()

packFixedRewards()
