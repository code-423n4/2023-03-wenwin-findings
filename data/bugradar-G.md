### Incrementing/Decrementing state variables inside loop 
- Lottery.sol; L125 : Loop in L125 increments three state variables (L193; L194; Ticket.sol L24). Would be better to refactor the incrementing to local variables and just update the state variables after the loop. especially for larger loops (many ticket buys) this should lead to good gas savings for one of your main customer entry points. https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L125; https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L24 
- Lottery.sol; L172: Loop in L172 decrements state variable at L266. Instead increment a local variable and set the state variable at the end of the loop. Logic of "claimWinningTicket" and "claimWinningTickets" can be put into one function anyways. https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L172, https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L266

### Unnecessary declaration of variables 
- Lottery.sol; L171: declaration of „totalTickets“ is waste of gas. https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L171

### Unnecessary external call due to "external" function  
- Lottery.sol; L261: Calling this.claimable externally wastes gas. Instead make the „claimable“ function public and not external. https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L261
