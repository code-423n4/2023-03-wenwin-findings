Here are several gas optimizable parts.
# `TicketUtils.isValidTicket`
We can optimize the function `TicketUtils.isValidTicket`.
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L17-L34
```js
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
            uint256 ticketSize;
            for (uint8 i = 0; i < selectionMax; ++i) {
                ticketSize += (ticket & uint256(1));
                ticket >>= 1;
            }
            return (ticketSize == selectionSize) && (ticket == uint256(0));
        }
    }
```
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


# `TicketUtils.reconstructTicket`
We can optimize the function `TicketUtils.reconstructTicket`.
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L63-L75
``` js
        bool[] memory selected = new bool[](selectionMax);

        for (uint256 i = 0; i < selectionSize; ++i) {
            uint8 currentNumber = numbers[i];
            // check current selection for numbers smaller than current and increase if needed
            for (uint256 j = 0; j <= currentNumber; ++j) {
                if (selected[j]) {
                    currentNumber++;
                }
            }
            selected[currentNumber] = true;
            ticket |= ((uint120(1) << currentNumber));
        }
```
The original time complexity is `O(selectionMax * selectionSize)`. (around 35 * 7)
We can optimize it to `O(selectionSize * selectionSize)`. (around 7 * 7)
Also, we can avoid using the boolean array with length of `selectionMax`.

The optimized code is here.
``` js
        if (selectionSize >= 2) {
            for (uint256 i = selectionSize - 2; i >= 0; --i) {
                for (uint256 j = selectionSize - 1; j > i; --j) {
                    if (numbers[i] <= numbers[j]) {
                        ++numbers[j];
                    }
                }
            }
        }

        for (uint256 i = 0; i < selectionSize; ++i) {
            ticket |= ((uint120(1) << numbers[i]));
        }
```

# `TicketUtils.ticketWinTier`
We can optimize the function `TicketUtils.ticketWinTier`.
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L83-L101
``` js
    function ticketWinTier(
        uint120 ticket,
        uint120 winningTicket,
        uint8 selectionSize,
        uint8 selectionMax
    )
        internal
        pure
        returns (uint8 winTier)
    {
        unchecked {
            uint120 intersection = ticket & winningTicket;
            for (uint8 i = 0; i < selectionMax; ++i) {
                winTier += uint8(intersection & uint120(1));
                intersection >>= 1;
            }
            assert((winTier <= selectionSize) && (intersection == uint256(0)));
        }
    }
```
The original complexity is `O(selectionMax)`. (around 35)
We can optimize it to `O(selectionSize)`. (around 7)

The optimized code is here.
``` js
    function ticketWinTier(
        uint120 ticket,
        uint120 winningTicket,
        uint8 selectionSize,
        uint8 selectionMax
    )
        internal
        pure
        returns (uint8 winTier)
    {
        unchecked {
            int256 intersection = int256(uint256(ticket & winningTicket));
            while (intersection > 0) {
                int256 i = intersection & (-intersection);
                intersection -= i;
                ++winTier;
            }
            assert((winTier <= selectionSize) && (intersection == int256(0)));
        }
    }
```