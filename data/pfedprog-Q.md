## Unsafe ERC20 Operation(s)

ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.

  https://github.com/code-423n4/2023-03-wenwin/src/staking/StakedTokenLock.sol::34 => stakedToken.transferFrom(msg.sender, address(this), amount);
  https://github.com/code-423n4/2023-03-wenwin/src/staking/StakedTokenLock.sol::47 => stakedToken.transfer(msg.sender, amount);
  https://github.com/code-423n4/2023-03-wenwin/src/staking/StakedTokenLock.sol::55 => rewardsToken.transfer(owner(), rewardsToken.balanceOf(address(this)));

## Unspecific Compiler Version Pragma

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

  ../2023-03-wenwin/src/VRFv2RNSource.sol::3 => pragma solidity ^0.8.7;
  ../2023-03-wenwin/src/interfaces/IVRFv2RNSource.sol::3 => pragma solidity ^0.8.7;
  ../2023-03-wenwin/src/staking/StakedTokenLock.sol::3 => pragma solidity ^0.8.17;