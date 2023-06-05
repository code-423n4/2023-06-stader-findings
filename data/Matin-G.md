1 - Cache the address of the `getPoolUtils()` once in the contract `UtilLib` and read that address instead of calling a view function in the `StaderConfig` contract:

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#123
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#124
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#148
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#149


`staderConfig.getPoolUtils()` in the contract `UtilLib` which is responsible to get the address of the pool utils from the `StaderConfig` contract, is used twice inside of the functions `getOperatorRewardAddress()` and `getValidatorSettleStatus()` in that contract. Caching the address at the first of the contract and reading from a memory variable will save gas instead of making a repetitive cross-contract view function call.