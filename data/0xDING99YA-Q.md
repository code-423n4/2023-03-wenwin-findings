## Title
Parameters for constructor of StakedTokenLock.sol lacks for necessary validation

## Proof of Concept
    IStaking public immutable override stakedToken;
    IERC20 public immutable override rewardsToken;
    uint256 public immutable override depositDeadline;
    uint256 public immutable override lockDuration;
    uint256 public override depositedBalance;

    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
        _transferOwnership(msg.sender);
        stakedToken = IStaking(_stakedToken);
        rewardsToken = stakedToken.rewardsToken();
        depositDeadline = _depositDeadline;
        lockDuration = _lockDuration;
    }

In the constructor, it doesn't check the validity of _stakedToken, _depositDeadline and _lockDuration. Since those variables are immutable it's necessary to check the validity to avoid undesired situation.

## Recommended Mitigation Steps
Make sure _stakedToken != address(0), _depositDeadline > 0, _lockDuration > 0 or use other requirements.