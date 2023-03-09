(1) Any person buying tickets can list themselves as the frontend and then claim rewards for the tickets sold, this way the can get back some of the money and buy the tickets at a discount.
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L129
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L151-L157
If this isn't intended, the only recommendation I can give is to register the frontends, only allowing the users buying the ticket to list the registered, verified frontends.

(2) Upon changing the oracle source, the  lastRequestTimestamp is not reset, so the contract will not be able to ask for any data for another duration of the maxRequestDelay (which is 5 hours)
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L64-L66
In this function the lastRequestTimestamp isn't reset: https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/RNSourceController.sol#L89-L104
I believe resetting the lastRequestTimestamp would make sense, because we changed the source of the data.

(3) Chainlink documentation states that fulfillRandomWords() shouldn't revert, but the in the code https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/VRFv2RNSource.sol#L32-L38 it reverts.
https://docs.chain.link/vrf/v2/security#fulfillrandomwords-must-not-revert
https://docs.chain.link/vrf/v2/security
