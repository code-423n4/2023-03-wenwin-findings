| Number | Optimization details                         |
| ------ | ---------------------------------------------|
| G-01   | no possible underflows, we can add unchecked{}|
| G-02   | using smaller uint value, we can use uint8 for MAX_MAX_FAILED_ATTEMPTS|

# [G-01] no possible underflows, we can add unchecked{}
`lottery.nextTicketId()` always will be greater than `lastUpdateTicketId`

### Recommended Steps to save gas
`` uint256 ticketsSoldSinceUpdate = unchecked{lottery.nextTicketId() - lastUpdateTicketId; }``


# [G-02] using smaller uint value, we can use uint8 for MAX_MAX_FAILED_ATTEMPTS
Save gas by using smaller uint value since the vars are constants, uint8 can go in place for `MAX_MAX_FAILED_ATTEMPTS` and the rest can be left for `MAX_REQUEST_DELAY` though it doesn’t need that much `uint256 private constant MAX_MAX_FAILED_ATTEMPTS = 10;` `uint256 private constant MAX_REQUEST_DELAY = 5 hours;`

### Recommended Steps to save gas
``uint8 private constant MAX_MAX_FAILED_ATTEMPTS = 10;``


