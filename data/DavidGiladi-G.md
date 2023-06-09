# Please note: I reported issues that were overlooked by the winning bot. I reported only on instances that were missed

#

## Optimal Struct Variable Order
- Severity: Gas Optimization
- Confidence: High

### Description
Detect optimal variable order in struct definitions to decrease the number of slots used.


- Optimization opportunity in struct `File: contracts/interfaces/INodeRegistry.sol#6-15` <br> [Validator](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/interfaces/INodeRegistry.sol#L6-L15)
- original variable order (count: 8 slots)
    - ValidatorStatus status
    - bytes pubkey
    - bytes preDepositSignature
    - bytes depositSignature
    - address withdrawVaultAddress
    - uint256 operatorId
    - uint256 depositBlock
    - uint256 withdrawnBlock


 - optimized variable order (count: 7 slots)
    - address withdrawVaultAddress
    - ValidatorStatus status
    - bytes pubkey
    - bytes preDepositSignature
    - bytes depositSignature
    - uint256 operatorId
    - uint256 depositBlock
    - uint256 withdrawnBlock


https://github.com/code-423n4/2023-06-stader/blob/main/contracts/interfaces/INodeRegistry.sol#L6-L15
#


</details>

# 


## Usage of Custom Errors for Gas Efficiency
- Severity: Gas Optimization
- Confidence: High

### Description
This detector flags functions that use revert()/require() strings, which are less gas efficient than custom errors. Custom errors, available from Solidity version 0.8.4, save approximately 50 gas each time they're used by avoiding the need to allocate and store the revert string.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- `File: contracts/VaultProxy.sol#47` <br> [revert(string(data))](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L47) use custom error instead 

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L47


</details>

# 




## Use Small Integer For Efficiency
- Severity: Gas Optimization
- Confidence: High

### Description
This detector flags contracts that inefficiently use `uint` or `int` of sizes smaller than 32 bytes. The EVM operates on 32 bytes at a time, thus using elements smaller than this may cause your contract's gas usage to be higher. Refer to the Solidity documentation for more details: https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html.

<details>

<summary>
There are 10 instances of this issue:

</summary>

###
- `File: contracts/PermissionlessNodeRegistry.sol#30` <br> [PermissionlessNodeRegistry.POOL_ID](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L30) `POOL_ID` use 256 bites instead of 8

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L30


- `File: contracts/PermissionedNodeRegistry.sol#31` <br> [PermissionedNodeRegistry.POOL_ID](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L31) `POOL_ID` use 256 bites instead of 8

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L31


- `File: contracts/VaultProxy.sol#12` <br> [VaultProxy.poolId](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L12) `poolId` use 256 bites instead of 8

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L12


- `File: contracts/PermissionedNodeRegistry.sol#33` <br> [PermissionedNodeRegistry.maxNonTerminalKeyPerOperator](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L33) `maxNonTerminalKeyPerOperator` use 256 bites instead of 64

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L33


- `File: contracts/PoolUtils.sol#13` <br> [PoolUtils.PUBKEY_LENGTH](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L13) `PUBKEY_LENGTH` use 256 bites instead of 64

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L13


- `File: contracts/PermissionlessNodeRegistry.sol#32` <br> [PermissionlessNodeRegistry.maxNonTerminalKeyPerOperator](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L32) `maxNonTerminalKeyPerOperator` use 256 bites instead of 64

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L32


- `File: contracts/PermissionlessNodeRegistry.sol#31` <br> [PermissionlessNodeRegistry.inputKeyCountLimit](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L31) `inputKeyCountLimit` use 256 bites instead of 16

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L31


- `File: contracts/PoolUtils.sol#14` <br> [PoolUtils.SIGNATURE_LENGTH](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L14) `SIGNATURE_LENGTH` use 256 bites instead of 64

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L14


- `File: contracts/PoolSelector.sol#18` <br> [PoolSelector.poolAllocationMaxSize](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L18) `poolAllocationMaxSize` use 256 bites instead of 16

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L18


- `File: contracts/PermissionedNodeRegistry.sol#32` <br> [PermissionedNodeRegistry.inputKeyCountLimit](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L32) `inputKeyCountLimit` use 256 bites instead of 16

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L32


</details>

# 
