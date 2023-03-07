# 1. requestId is unique by design, no need to check

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L22-L25

```
        uint256 requestId = requestRandomnessFromUnderlyingSource();
        if (requests[requestId].status != RequestStatus.None) {
            revert requestIdAlreadyExists(requestId);
        }
```

As the requestId given by ChainLink is a hash including incrementing nonce:

https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/VRFCoordinatorV2.sol#L410-L411

```
requestId is a hash of (keyHash, msg.sender, subId, nonce);
in which nonce is incremented for every request
```

So, it's guaranteed that requestId is unique, we can remove this `requests[requestId].status != RequestStatus.None` check to save gas