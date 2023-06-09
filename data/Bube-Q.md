[N-01] Non-library/interface files should use fixed compiler versions, not floating ones

Avoid floating pragmas for non-library contracts. While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain. It is recommended to pin to a concrete compiler version.

There is one more instance of this issue (the other instance is known issue):

File: contracts/interfaces/IVaultProxy.sol

    2: pragma solidity ^0.8.16;

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/interfaces/IVaultProxy.sol#L2