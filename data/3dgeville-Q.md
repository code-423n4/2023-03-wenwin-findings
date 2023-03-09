https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/Staking.sol#L94

instead of >  should be >=

https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/staking/Staking.sol#L79-L89

Should burn after transfer 

https://github.com/code-423n4/2023-03-wenwin/blob/940066dc3c500cf745afc9e9381e131da7f98e88/src/Ticket.sol#L23-L27

Should use safe mint

Furthermore the function has a callback to the "to" address argument. Functions with callbacks should have reentrancy guards in place for protection against possible malicious actors both from inside and outside the protocol. Add a reentrancy guard modifier on the mint() function (Nonreentrant)