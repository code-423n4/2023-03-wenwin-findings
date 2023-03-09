1 - It is not necessary to calculate again the tickets sold by draw.
==

The next line [uint256 ticketsSoldDuringDraw = nextTicketId - lastDrawFinalTicketId;](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L227) calculates how many tickets were sold during the draw but the [ticketSold array](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L39) has the number of tickets sold by draw. The ```ticketSold``` is used in the same function [ticketsSold[drawFinalized]](https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L218) so it is possible to use ```ticketSold``` instead.

Recalculate the ticket sold value is not necessary, use ```ticketSold``` instead and save gas.

