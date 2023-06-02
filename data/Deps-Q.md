# Missing __gap[] in VaultFactory upgradeable contract

Problem:
The contract placed in contracts/factory/VaultFactory.sol inside the repo does not have a `__gap[]` at the beginning and at the end of the contract.

Impact:
Possible storage collision when/if proxy contract gets upgraded, causing possible loss in the following variables.


``IStaderConfig public staderConfig;``
``address public vaultProxyImplementation;``
``bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');``


