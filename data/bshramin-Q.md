## Title
Draws can only be executed sequentially


## Links to affected code:
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L133-L142


## Impact
* Excess calls to the Oracle
* Excess gas consumption
* The probability of no one wanting to call an execution:
* If no one uses the contract for some amount of time it will become less and less interesting to use, making the comeback impossible after some time.


## Proof of Concept
The following test shows the problem in action.

```solidity
function testSequentialExecution() public {
    vm.startPrank(USER);
    rewardToken.mint(2*TICKET_PRICE);
    rewardToken.approve(address(lottery), 2*TICKET_PRICE);
    uint256 pastDrawWinningTicketId = buyTicket(lottery.currentDraw(), uint120(0xF0), address(0));
    uint256 currentDrawWinningTicketId = buyTicket(lottery.currentDraw()+5, uint120(0xF0), address(0));
    vm.stopPrank();

    vm.warp(block.timestamp + PERIOD * 7);

    uint256 randomNumber = 0xF0;
    lottery.executeDraw();
    vm.prank(randomNumberSource);
    lottery.onRandomNumberFulfilled(randomNumber);

    vm.startPrank(USER);
    claimTicket(pastDrawWinningTicketId); // Succeeds
    vm.expectRevert(abi.encodeWithSelector(NothingToClaim.selector, currentDrawWinningTicketId));
    claimTicket(currentDrawWinningTicketId); // Fails but could succeed if the right execution logic was implemented
    vm.stopPrank();


    for (uint i; i< 6;++i) {
        lottery.executeDraw();
        vm.prank(randomNumberSource);
        lottery.onRandomNumberFulfilled(randomNumber);
    }

    vm.prank(USER);
    claimTicket(currentDrawWinningTicketId); // Succeeds
}
```


![image](https://i.ibb.co/4W6N5SN/sequential.jpg)


## Tools Used
Manual Review


## Recommended Mitigation Steps
Some possible solutions:
* Execute draw can execute all the draws that can be executed, instead of just the next one
* Execute draw can take a drawId and only execute that particular draw
* We can at least mitigate this problem by returning the correct error message from `function claimable` when the draw is not executed yet. This way the user will know that the draw is not executed yet and will not try to claim the ticket.