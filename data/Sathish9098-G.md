# GAS OPTIMIZATIONS

All the gas optimizations are determined based opcodes and sample tests.

| Count | Issues | Instances | Gas Saved |
|----------|----------|----------|----------|
| [G-1]   | Using bools for storage incurs overhead   | 7   | 119700  |
| [G-2]   | Using storage instead of memory for structs/arrays saves gas   | 4   | 8400  |
| [G-3]   | External function calls should be cached outside the loop    | 3   | PerCall (2100 gas) |
| [G-4]   | Avoid emitting storage variable when stack variable available    | 4   | 400   |
| [G-5]   | Refactor the for loop to avoid unwanted variable creation inside the loop   | 1   | 13   |
| [G-6]   | Cache the state variable out side the loop  | 1  | PerCall (100 gas)   |
| [G-7]   | Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)   | 3   | 39   |
| [G-8]   | Repack the state variable to avoid on extra slot    | 1   | 2000   |


##

## [G-1] Using bools for storage incurs overhead

### All instances are not found by bot race

Results : ``INSTANCES 7``, Saves ``119700 GAS`` 

```
   // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

```

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27 Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#LL54C4-L54C61

```solidity
FILE: Breadcrumbs2023-06-stader/contracts/PermissionedNodeRegistry.sol

54: mapping(address => bool) public override permissionList;

```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L18-L19

```solidity
FILE: 2023-06-stader/contracts/StaderOracle.sol

18: bool public override erInspectionMode;
19: bool public override isPORFeedBasedERData;

44:  mapping(address => bool) public override isTrustedNode;
45:  mapping(bytes32 => bool) private nodeSubmissionKeys;

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L10-L11

```solidity
FILE: 2023-06-stader/contracts/VaultProxy.sol

10: bool public override isValidatorWithdrawalVault;
11: bool public override isInitialized;

```

##

## [G-2] Using storage instead of memory for structs/arrays saves gas

### All instances are not found by bot race

Results : ``INSTANCES 3``, Saves ``8400 GAS`` 

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a ``Gcoldsload (2100 gas)`` for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

### Using ``storage`` variables can potentially save gas by ``eliminating the cost of copying the struct from storage to memory``

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L733

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L739

### Saves ``4200 GAS``

```diff
FILE: 2023-06-stader/contracts/PermissionedNodeRegistry.sol

- 733:  Validator memory validator = validatorRegistry[_validatorId];
+ 733:  Validator storage validator = validatorRegistry[_validatorId];


- 739:  Validator memory validator = validatorRegistry[_validatorId];
+ 739:  Validator storage validator = validatorRegistry[_validatorId];

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#LL692C9-L692C70

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L702

### Saves ``4200 GAS``

```diff
FILE: 2023-06-stader/contracts/PermissionlessNodeRegistry.sol

- 692: Validator memory validator = validatorRegistry[_validatorId];
+ 692: Validator storage validator = validatorRegistry[_validatorId];

- 702: Validator memory validator = validatorRegistry[_validatorId];
+ 702: Validator storage validator = validatorRegistry[_validatorId];
```
##

## [G-3] External function calls should be cached outside the loop 

Results : ``INSTANCES 3``, Saves ``2100 GAS per call`` 

Results : Saves Approximately  ``2100 gas`` for single call 

When caching external call values, it's generally more efficient to perform the external call outside of any loops to avoid unnecessary redundant calls.

In terms of opcode savings, caching the external call result outside the loop can eliminate repeated opcodes such as ``CALL`` that would have been used for the external calls within the loop. This can reduce the overall gas consumption.

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderStakePoolsManager.sol#L231-L239

### ``staderConfig.getStakedEthPerNode()`` external call should be cached outside the loop to avoid redundant calls saves at least ``700 GAS`` for every iterations.

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

## [G-4] Avoid emitting storage variable when stack variable available 

Results : ``INSTANCES 4``, Saves ``400 GAS `` 

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

## [G-5] Refactor the for loop to avoid unwanted variable creation inside the loop

Results : ``INSTANCES 1``, Saves ``13 GAS per call `` 

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

## [G-6] Cache the state variable out side the loop

Results : ``INSTANCES 1``, Saves ``100 GAS per call `` 

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Penalty.sol#L100-L101

### staderConfig value should be cached with local stack variable outside the loops. For every iteration this saves ``1 SLOD 100 GAS``

```diff
FILE: 2023-06-stader/contracts/Penalty.sol
+ IStaderConfig public _staderConfig = staderConfig ;
100: for (uint256 i; i < reportedValidatorCount; ) {
101:            if (UtilLib.getValidatorSettleStatus(_pubkey[i], _staderConfig)) {

```

##

## [G-7] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

Results : ``INSTANCES 3``, Saves ``39 GAS `` 

Caching global variable consumes extra 13 gas 

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L59

### ``msg.sender`` cache global variables is more expensive. This cost extra ``13 GAS`` 

```diff
FILE: Breadcrumbs2023-06-stader/contracts/SDCollateral.sol

INSTANCE 1: 

- 59:  address operator = msg.sender;
- 60:        uint256 opSDBalance = operatorSDBalance[operator];
+ 60:        uint256 opSDBalance = operatorSDBalance[msg.sender];
61:
- 62:        if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
+ 62:        if (opSDBalance < getOperatorWithdrawThreshold(msg.sender) + _requestedSD) {
63:            revert InsufficientSDToWithdraw(opSDBalance);
64:        }
- 65:        operatorSDBalance[operator] -= _requestedSD;
+ 65:        operatorSDBalance[msg.sender] -= _requestedSD;
66:
67:        // cannot use safeERC20 as this contract is an upgradeable contract
- 68:        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
+ 68:        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(msg.sender), _requestedSD)) {
69:            revert SDTransferFailed();
70:        }
71:
- 72:        emit SDWithdrawn(operator, _requestedSD);
+ 72:        emit SDWithdrawn(msg.sender, _requestedSD);
73:    }

INSTANCE: 2:

- 44:  address operator = msg.sender;
- 45:        operatorSDBalance[operator] += _sdAmount;
+ 45:        operatorSDBalance[msg.sender] += _sdAmount;
46:
- 47:        if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {
+ 47:        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
48:            revert SDTransferFailed();
49:        }
50:
- 51:        emit SDDeposited(operator, _sdAmount);
+ 51:        emit SDDeposited(msg.sender, _sdAmount);

```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#L113

### ``msg.sender`` cache global variables is more expensive. This cost extra ``13 GAS`` 

```diff
FILE: Breadcrumbs2023-06-stader/contracts/SocializingPool.sol

- 113: address operator = msg.sender;
- 114:        (uint256 totalAmountSD, uint256 totalAmountETH) = _claim(_index, operator, _amountSD, _amountETH, _merkleProof);
+ 114:        (uint256 totalAmountSD, uint256 totalAmountETH) = _claim(_index, msg.sender, _amountSD, _amountETH, _merkleProof);

- 116:        address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(operator, staderConfig);
+ 116:        address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig);

```
##

## [G-8] Repack the state variable to avoid on extra slot 

Results : ``INSTANCES 1``, Saves ``2000 GAS `` 

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

[_erChangeLimit > ER_CHANGE_MAX_BPS](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L532) as per this condition check the ``erChangeLimit`` not exceeds ``10000``. So ``uint64`` alone more than enough instead of uint256
 
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L18-L28

### Saves 1 slot and 2000 gas 

```diff
FILE: 2023-06-stader/contracts/StaderOracle.sol

18:    bool public override erInspectionMode;
19:    bool public override isPORFeedBasedERData;
20:    SDPriceData public lastReportedSDPriceData;
+ 28:    uint64 public override erChangeLimit;
21:    IStaderConfig public override staderConfig;
22:    ExchangeRate public inspectionModeExchangeRate;
23:    ExchangeRate public exchangeRate;
24:    ValidatorStats public validatorStats;
25:
26:    uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days
27:    uint256 public constant ER_CHANGE_MAX_BPS = 10000;
- 28:    uint256 public override erChangeLimit;

```
