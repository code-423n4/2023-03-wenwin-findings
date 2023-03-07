We can optimize the function `TicketUtils.isValidTicket`.
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L17-L34

The original complexity is `O(selectionMax)`. (around 35)
We can optimize it to `O(selectionSize)`. (around 7)

The optimized code is here.
``` js
    function isValidTicket(
        uint256 ticket,
        uint8 selectionSize,
        uint8 selectionMax
    )
        internal
        pure
        returns (bool isValid)
    {
        unchecked {
            if ((ticket >> selectionMax) != 0) {
                return false;
            }
            int256 _ticket = int256(ticket);
            uint8 ticketSize;
            while (_ticket > 0) {
                int256 i = _ticket & (-_ticket);
                _ticket -= i;
                if (++ticketSize > selectionSize) {
                    return false;
                }
            }
            return ticketSize == selectionSize && _ticket == 0;
        }
    }
```

We have a trick for iterating only valid bits.