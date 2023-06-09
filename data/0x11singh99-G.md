# [G-01] Memory variables can be cached to stack if reading and writing happening multiple times

Memory variables should be cached to stack variables if re-reading happening of same variables more than once, here for each iteration it will save 100 gas almost.
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L210](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L210)
```solidity
File : contracts/PermissionedNodeRegistry.sol

                //@audit-issue gas optimization if used stack over memory variable for each iteration     
       210:      remainingOperatorCapacity[i] = getOperatorQueuedValidatorCount(i);
       211:      selectedOperatorCapacity[i] = Math.min(remainingOperatorCapacity[i], validatorPerOperator);
                //@audit gas
       212:      totalValidatorToDeposit += selectedOperatorCapacity[i];
                //@audit gas
       213:      remainingOperatorCapacity[i] -= selectedOperatorCapacity[i];
       214:      unchecked {
                    ++i;
                }
        
```
Converting above to below code save more than 100 gas for each loop iteration
Take 2 variables for caching from memory
``` solidity
                uint256 a;
                uint256 b;
                
                a = remainingOperatorCapacity[i];
                b = selectedOperatorCapacity[i];
               
                a = getOperatorQueuedValidatorCount(i);
                b = Math.min(a, validatorPerOperator);
                //@audit gas,converting <x>+=<y> -> <x>=<x>+<y> costs less gas
                totalValidatorToDeposit = totalValidatorToDeposit + b;
                //@audit gas,converting <x>-=<y> -> <x>=<x>-<y> costs less gas
                a = a-b; 
                unchecked {
                    ++i;
                }
                remainingOperatorCapacity[i] = a;
                selectedOperatorCapacity[i] = b;

```
# [G-02]  <x>=<x>+<y> and <x>=<x>-<y> costs less gas compare to <x>+=<y> and <x>-=<y>
### There are 4 instances of this
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L212](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L212)
```solidity
File: contracts/PermissionedNodeRegistry.sol
 212:   totalValidatorToDeposit += selectedOperatorCapacity[i];
```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L213](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L213)
```solidity
File: contracts/PermissionedNodeRegistry.sol
213:   remainingOperatorCapacity[i] -= selectedOperatorCapacity[i];
```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L234](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L234)
```solidity
File: contracts/PermissionedNodeRegistry.sol
234:   selectedOperatorCapacity[i] += newSelectedCapacity;
```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L235](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L235)
```solidity
File: contracts/PermissionedNodeRegistry.sol
235:  remainingValidatorsToDeposit -= newSelectedCapacity;
```

# [G-03]  Calldata array elements  should be cached in stack variables rather than re-reading them from calldata array
Cache calldata elements into stack variables if using same elements multiple times it save gas.
## PermissionedNodeRegistry.sol.addValidatorKeys() : inside loop for each  *i*  value _pubkey[i], _preDepositSignature[i], _depositSignature[i] should be cached to stack variables (saves 412 gas on each iteration)
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L156](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L156)
``` solidity
File : contracts/PermissionedNodeRegistry.sol 
   156:   IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);
            address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(
                POOL_ID,
                operatorId,
                operatorTotalKeys + i, //operator totalKeys
                nextValidatorId
            );
            validatorRegistry[nextValidatorId] = Validator(
                ValidatorStatus.INITIALIZED,
                _pubkey[i],
                _preDepositSignature[i],
                _depositSignature[i],
                withdrawVault,
                operatorId,
                0,
                0
            );
            validatorIdByPubkey[_pubkey[i]] = nextValidatorId;
            validatorIdsByOperatorId[operatorId].push(nextValidatorId);
            emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId);
```
## PermissionedNodeRegistry.sol.markValidatorReadyToDeposit() : inside loop for each  *i*  value  _frontRunPubkey[i] should be cached to stack variables (saves 25 gas on each iteration)
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L273](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L273)
```solidity
272:   for (uint256 i; i < frontRunValidatorsLength; ) {
273:           uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];
          ...
278:         emit ValidatorMarkedAsFrontRunned(_frontRunPubkey[i], validatorId); //@audit-gas _frontRunPubkey[i] can be cached 
             ... }
```
## PermissionedNodeRegistry.sol.markValidatorReadyToDeposit() : inside loop for each  *i*  value  _invalidSignaturePubkey[i] should be cached to stack variables (saves 25 gas on each iteration)
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L273](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L273)
```solidity
285:    for (uint256 i; i < invalidSignatureValidatorsLength; ) {
286:       uint256 validatorId = validatorIdByPubkey[_invalidSignaturePubkey[i]];
           ...

291:       emit ValidatorStatusMarkedAsInvalidSignature(_invalidSignaturePubkey[i], validatorId);
           ...
        }
```

# [G-04]  State variables  should be cached in stack variables rather than re-reading them from storage.
If reading of state variables is happening more than once inside function than it must be cached in stack variables so 100 Gas of Gwarmaccess can be save every time a state variables read after first time.

## PermissionedNodeRegistry.sol.addValidatorKeys() : state variable *nextValidatorId* should be cached into stack varaible insted of re-reding from storage 7 times (saves 600 Gas)
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L161](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L161)
```solidity
          address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(
                POOL_ID,
                operatorId,
                operatorTotalKeys + i, //operator totalKeys
   161:             nextValidatorId
            );
   163:        validatorRegistry[nextValidatorId] = Validator(
                ValidatorStatus.INITIALIZED,
                _pubkey[i],
                _preDepositSignature[i],
                _depositSignature[i],
                withdrawVault,
                operatorId,
                0,
                0
            );
   173:      validatorIdByPubkey[_pubkey[i]] = nextValidatorId;
   174:      validatorIdsByOperatorId[operatorId].push(nextValidatorId);
   175:      emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId);
   176:      nextValidatorId++;
```

## PermissionedNodeRegistry.sol.allocateValidatorsAndUpdateOperatorId() : state variable *nextOperatorId* should be cached into stack varaible insted of re-reding from storage 3 times + no. of iterations of for loop (saves 200 + 100*n Gas of Gwarmaccess, n is the for loop iterations) 
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L199](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L199)
```solidity
 199:    selectedOperatorCapacity = new uint256[](nextOperatorId);//@audit -gas
         uint256 validatorPerOperator = _numValidators / totalActiveOperatorCount;
 202:    uint256[] memory remainingOperatorCapacity = new uint256[](nextOperatorId);//@audit -gas
         uint256 totalValidatorToDeposit;
         bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);
         if (validatorPerOperatorGreaterThanZero) {
                         //@audit -gas
 206:        for (uint256 i = 1; i < nextOperatorId; ) {
                if (!operatorStructById[i].active) {
                    continue;
                }
             ... 

 223:    uint256 totalOperators = nextOperatorId - 1;//@audit -gas
```
## PermissionedNodeRegistry.sol.allocateValidatorsAndUpdateOperatorId() : state variable *operatorIdForExcessDeposit* should be cached into stack varaible insted of re-reding from storage no. of iterations of do-while loop (100*n - 100 Gas of Gwarmaccess, n is the do-while loop iterations) 
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L241](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L241)
```solidity
241:         } while (i != operatorIdForExcessDeposit); //audit gas  cache it rather than re-reading same state variable
```

# [G-05]  Make instance into stack variable instead of re-creating every time calling external contract function (save 30 gas each time )

In most files this pattern is not followed, this should be followed.

