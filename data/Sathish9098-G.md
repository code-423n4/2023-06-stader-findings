# GAS OPTIMIZATIONS

## [G-] External calls should be cached outside the loop 

When caching external call values, it's generally more efficient to perform the external call outside of any loops to avoid unnecessary redundant calls.

In terms of opcode savings, caching the external call result outside the loop can eliminate repeated opcodes such as ``CALL, STATICCALL, or DELEGATECALL`` that would have been used for the external calls within the loop. This can reduce the overall gas consumption.

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderStakePoolsManager.sol#L231-L239

### ``staderConfig.getStakedEthPerNode()`` external call should be cached outside the loop to avoid redundant calls saves ``700 GAS`` for every iterations.

```diff

FILE: 2023-06-stader/contracts/StaderStakePoolsManager.sol

+  uint256 stakedEthPerNode=staderConfig.getStakedEthPerNode();
231: uint256 poolCount = poolIdArray.length;
232:        for (uint256 i = 0; i < poolCount; i++) {
233:            uint256 validatorToDeposit = selectedPoolCapacity[i];
235:            if (validatorToDeposit == 0) {
236:                continue;
237:            }
238:            address poolAddress = IPoolUtils(poolUtils).poolAddressById(poolIdArray[i]);
- 239:           uint256 poolDepositSize = staderConfig.getStakedEthPerNode() -
+ 239:           uint256 poolDepositSize = stakedEthPerNode -
240:                IPoolUtils(poolUtils).getCollateralETH(poolIdArray[i]);

```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessPool.sol#L94-L99

### ``INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID()`` external call should be cached outside the loop to avoid redundant calls saves ``1400 GAS`` for every iterations.

```diff
FILE: Breadcrumbs2023-06-stader/contracts/PermissionlessPool.sol

+ uint8 poolId= INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID() ;
94: for (uint256 i; i < pubkeyCount; ) {
95:            address withdrawVault = IVaultFactory(vaultFactory).computeWithdrawVaultAddress(
- 96:                INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID(),
+ 96:                poolId,
97:                _operatorId,
98:                _operatorTotalKeys + i
99:            );

```
##

## [G-] Avoid emitting storage variable when stack variable available 

Results : Saves ``5 SLOD``, ``500 gas`` 

- When accessing a storage variable, an opcode like SLOAD is used to read the value from storage into the stack. This incurs a gas cost based on the complexity of accessing storage and the amount of data being loaded

- Stack variables, on the other hand, do not require the use of SLOAD or SSTORE opcodes. They are directly manipulated within the function's stack frame, which is a more efficient operation in terms of gas usage

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L147-L148

### ``_duration`` stack variable should be used instead of storage variable ``duration`` saves ``1 SLOD (100 gas ) ``

```diff
FILE: 2023-06-stader/contracts/Auction.sol

147:  duration = _duration;
- 148:        emit AuctionDurationUpdated(duration);
+ 148:        emit AuctionDurationUpdated(_duration);
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L270-L271

### ``_nextQueuedValidatorIndex`` stack variable should be used instead of storage variable ``nextQueuedValidatorIndex `` saves ``1 SLOD (100 gas )`` 

```diff
FILE: Breadcrumbs2023-06-stader/contracts/PermissionlessNodeRegistry.sol

270:    nextQueuedValidatorIndex = _nextQueuedValidatorIndex;
- 271:        emit UpdatedNextQueuedValidatorIndex(nextQueuedValidatorIndex);
+ 271:        emit UpdatedNextQueuedValidatorIndex(_nextQueuedValidatorIndex);
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L338-L339

### ``_maxNonTerminalKeyPerOperator`` stack variable should be used instead of storage variable ``maxNonTerminalKeyPerOperator `` saves ``1 SLOD (100 gas )`` 

```diff
FILE: Breadcrumbs2023-06-stader/contracts/PermissionlessNodeRegistry.sol

338: maxNonTerminalKeyPerOperator = _maxNonTerminalKeyPerOperator;
- 339:        emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);
+ 339:        emit UpdatedMaxNonTerminalKeyPerOperator(_maxNonTerminalKeyPerOperator);
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L535-L536

###  ``_erChangeLimit`` stack variable should be used instead of storage variable ``erChangeLimit `` saves ``1 SLOD (100 gas )`` 

```diff 

FILE: Breadcrumbs2023-06-stader/contracts/StaderOracle.sol

535: erChangeLimit = _erChangeLimit;
- 536:         emit UpdatedERChangeLimit(erChangeLimit);
+ 536:         emit UpdatedERChangeLimit(_erChangeLimit);
```

##

## [G-] Refactor the for loop to avoid unwanted variable creation inside the loop

The variable pubkeyRoot is declared within each iteration of the for loop, but it is only used once. This results in unnecessary variable creation.This refactoring helps to optimize gas costs and code readability by reducing the number of variable declarations within the loop, especially for variables that are only used once.

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L485-L490

### ``UtilLib.getPubkeyRoot(_mapd.sortedPubkeys[i])`` can apply value directly instead of cache with local variable. For every iteration can saves ``15-20 GAS ``

```diff
FILE: 2023-06-stader/contracts/StaderOracle.sol

485: for (uint256 i; i < keyCount; ) {
- 486:                 bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_mapd.sortedPubkeys[i]);
- 487:                missedAttestationPenalty[pubkeyRoot]++;
+ 487:                missedAttestationPenalty[UtilLib.getPubkeyRoot(_mapd.sortedPubkeys[i])]++;
```

##

## [G-] Cache the state variable out side the loop

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Penalty.sol#L100-L101

### staderConfig value should be cached with local stack variable outside the loops. For every iteration this saves ``1 SLOD 100 GAS``

```diff
FILE: 2023-06-stader/contracts/Penalty.sol
+ IStaderConfig public _staderConfig = staderConfig ;
100: for (uint256 i; i < reportedValidatorCount; ) {
101:            if (UtilLib.getValidatorSettleStatus(_pubkey[i], _staderConfig)) {

```

## [G-] Using storage instead of memory for structs/arrays saves gas

##

## [G-] Use calldata instead of memory 

##
 
## [G-] State variables can be packed into fewer storage slots	2	-

##

## [G-] [G‑06]	Structs can be packed into fewer storage slots	2	-

##

## [G-] Values only set in the constructor should be immutable 

##

## [G-] Bool incurs overhead 

##

## [G-] Avoid initialize default values 





[G‑01]	Reduce gas usage by moving to Solidity 0.8.19 or later	47	-
[G‑02]	Avoid updating storage when the value hasn't changed	21	16800
[G‑03]	Remove or replace unused state variables	1	-
[G‑04]	Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate	2	-
[G‑05]	State variables can be packed into fewer storage slots	2	-
[G‑06]	Structs can be packed into fewer storage slots	2	-
[G‑07]	Using storage instead of memory for structs/arrays saves gas	1	4200
[G‑08]	State variables should be cached in stack variables rather than re-reading them from storage	181	17557
[G‑09]	Multiple accesses of a mapping/array should use a local variable cache	18	756
[G‑10]	The result of function calls should be cached rather than re-calling the function	12	-
[G‑11]	<x> += <y> costs more gas than <x> = <x> + <y> for state variables	12	1356
[G‑12]	internal functions only called once can be inlined to save gas	32	640
[G‑13]	Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement	1	85
[G‑14]	++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops	16	960
[G‑15]	Optimize names to save gas	38	836
[G‑16]	Using bools for storage incurs overhead	5	85500
[G‑17]	>= costs less gas than >	7	21
[G‑18]	++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)	36	180
[G‑19]	Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead	6	-
[G‑20]	Using private rather than public for constants, saves gas	35	-
[G‑21]	Don't use SafeMath once the solidity version is 0.8.0 or greater	2	-
[G‑22]	Division by two should use bit shifting	6	120
[G‑23]	Superfluous event fields	13	-
[G‑24]	Functions guaranteed to revert when called by normal users can be marked payable	71	1491
[G‑25]	Constructors can be marked payable	21	441
[G‑26]	Not using the named return variables anywhere in the function is confusing	7	-