At: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L17
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L34
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L47
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L73
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L74
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L76
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L86
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L88
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L92
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L93
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L95
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L98
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L99
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L104
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L70
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L71
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L107
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L128
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L130
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L146
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L152
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L175
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L192
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L195
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L268
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L18
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L19
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L23
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L18
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L19
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L29
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L47
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L81
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L139
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L142
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L145
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L148
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L152

- Use `_msgSender()` from `@openzeppelin/contracts/utils/Context.sol` instead of `msg.sender`

At: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L121
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L122
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L124
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L125
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L128
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L129
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L130
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L172

- Store `drawIds.length`, `tickets.length` in stack (#L121)

`uint256 drawIdsLength = drawIds.length;
 uint256 ticketsLength = tickets.length;
 if (drawIdsLength != ticketsLength) {`

- Store `drawIds.length`, `tickets.length` in stack (#L122)

`uint256 drawIdsLength = drawIds.length;
 uint256 ticketsLength = tickets.length;
 revert DrawsAndTicketsLenMismatch(drawIdsLength, ticketsLength);`

- Store `tickets.length` in stack (#L124)

`uint256 ticketsLength = tickets.length;
 ticketIds = new uint256[](ticketsLength);`

- Store `drawIds.length`, `tickets.length` in stack (#L125)

`uint256 drawIdsLength = drawIds.length;
 for (uint256 i; i < drawIdsLength;) {`

- Optimize for loop (#L125)

`for (uint256 i; i < drawIdsLength;) {
        ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
        unchecked { ++i; }
     }`

- Store `tickets.length` in stack (#L128)

`uint256 ticketsLength = tickets.length;
 referralRegisterTickets(currentDraw, referrer, _msgSender(), ticketsLength);`

- Store `tickets.length` in stack (#L129)

`uint256 ticketsLength = tickets.length;
 frontendDueTicketSales[frontend] += ticketsLength;`

- Store `tickets.length` in stack (#L130)

`uint256 ticketsLength = tickets.length;
 rewardToken.safeTransferFrom(_msgSender(), address(this), ticketPrice * ticketsLength);`

- Optimize for loop (#L172)

`for (uint256 i; i < totalTickets;) {
        claimedAmount += claimWinningTicket(ticketIds[i]);
        unchecked { ++i; }
     }`

At: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L32
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L35
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L77

- Store `_rewardsToReferrersPerDraw.length` in stack (#L32)

`uint256 rewardsToReferrersPerDrawLength = _rewardsToReferrersPerDraw.length;
 if (rewardsToReferrersPerDrawLength == 0) {`

- Store `_rewardsToReferrersPerDraw.length` in stack (#L35)

`uint256 rewardsToReferrersPerDrawLength = _rewardsToReferrersPerDraw.length;
 for (uint256 i; i < rewardsToReferrersPerDrawLength;) {`

- Optimize for loop (#L35)

`for (uint256 i; i < rewardsToReferrersPerDrawLength;) {
        if (_rewardsToReferrersPerDraw[i] == 0) {
            revert ReferrerRewardsInvalid();
        }
        unchecked { ++i; }
     }`

- Store `drawIds.length` in stack (#L77)

`uint256 drawIdsLength = drawIds.length;
 for (uint256 counter; counter < drawIdsLength;) {`

- Optimize for loop (#L77)

`for (uint256 counter; counter < drawIdsLength; ++counter) {
        claimedReward += claimPerDraw(drawIds[counter]);
        unchecked { ++counter; }
     }`

At: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L28
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L57
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L65
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L68
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L95

- Remove explicit `i` initialization (#L28)

`for (uint8 i; i < selectionMax; ++i) {`

- Optimize for loop (#L57)

`for (uint256 i; i < selectionSize;) {
        numbers[i] = uint8(randomNumber % currentSelectionCount);
        randomNumber /= currentSelectionCount;
        currentSelectionCount--;
        unchecked { ++i; }
     }`

- Optimize un-nested for loop (#L65)

`for (uint256 i; i < selectionSize;) {
     uint8 currentNumber = numbers[i];
     // check current selection for numbers smaller than current and increase if needed
     for (uint256 j; j <= currentNumber;) {
         if (selected[j]) {
             currentNumber++;
         }
         unchecked { ++j; }
     }
     selected[currentNumber] = true;
     ticket |= ((uint120(1) << currentNumber));
     unchecked { ++i; }
 }`

- Optimize nested for loop (#L68)

` for (uint256 j; j <= currentNumber;) {
      if (selected[j]) {
          currentNumber++;
      }
      unchecked { ++j; }
  }`

- Remove explicit `i` initialization (#L95)

`for (uint8 i; i < selectionMax; ++i) {
         winTier += uint8(intersection & uint120(1));
         intersection >>= 1;
     }`

At: https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L33
    https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L34

- Store `randomWords.length` in stack (#L33)

`uint256 randomWordsLength = randomWords.length;
 if (randomWordsLength != 1) {`

- Store `randomWords.length` in stack (#L34)

`uint256 randomWordsLength = randomWords.length;
 revert WrongRandomNumberCountReceived(requestId, randomWordsLength);`
