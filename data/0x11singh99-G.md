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

# [G-03]  State variables  should be cached in stack variables rather than re-reading them from storage.
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
## PermissionLessNodeRegistry.sol.onboardNodeOperator() : state variable *nextOperatorId* should be cached into stack varaible insted of re-reding from storage 2 times (saves 100 Gas of 2nd Gwarmaccess)
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L108](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L108)
```solidity
 106:      address nodeELRewardVault = IVaultFactory(staderConfig.getVaultFactory()).deployNodeELRewardVault(
            POOL_ID,
 108:          nextOperatorId
              );
 110:      nodeELRewardVaultByOperatorId[nextOperatorId] = nodeELRewardVault;
```
## PermissionLessNodeRegistry.sol.getNodeELVaultAddressForOptOutOperators() : state variable *nextOperatorId* should be cached into stack varaible insted of re-reding from storage 2 times (saves 100 Gas of 2nd Gwarmaccess)
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L570](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L570)
```solidity
570:   endIndex = endIndex > nextOperatorId ? nextOperatorId : endIndex; // cache nextOperatorId
```
## PermissionLessNodeRegistry.sol.onboardOperator() : state variable *nextOperatorId* should be cached into stack varaible insted of re-reding from storage 5 times (saves 400 Gas of  Gwarmaccess)
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L570](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L570)
```solidity
603:    operatorStructById[nextOperatorId] = Operator(
            true,
            _optInForSocializingPool,
            _operatorName,
            _operatorRewardAddress,
            msg.sender
           );
610:     operatorIDByAddress[msg.sender] = nextOperatorId;
611:     socializingPoolStateChangeBlock[nextOperatorId] = block.number;
612:     nextOperatorId++;

614:     emit OnboardedOperator(msg.sender, _operatorRewardAddress, nextOperatorId - 1, _optInForSocializingPool);
```
## PermissionLessNodeRegistry.sol.addValidatorKeys() : state variable *nextValidatorId* should be cached into stack varaible insted of re-reding from storage 6*n times (saves 600*n - 100 Gas of  Gwarmaccess,n is the for loop iterations ie. keyCount)
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L146](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L146)
```solidity
 140:    for (uint256 i; i < keyCount; ) {
            IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);
            address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(
                POOL_ID,
                operatorId,
                operatorTotalKeys + i, //operator totalKeys
 146:            nextValidatorId //@audit-gas cache it
            );
             //@audit-gas cache nextValidatorId
 148:         validatorRegistry[nextValidatorId] = Validator(
                ValidatorStatus.INITIALIZED,
                _pubkey[i],
                _preDepositSignature[i],
                _depositSignature[i],
                withdrawVault,
                operatorId,
                0,
                0
            );

 159:       validatorIdByPubkey[_pubkey[i]] = nextValidatorId;//@audit-gas cache  nextValidatorId
 160:       validatorIdsByOperatorId[operatorId].push(nextValidatorId);//@audit-gas cache nextValidatorId
 161:       emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId);//@audit-gas cache  nextValidatorId
 162:       nextValidatorId++; //@audit-gas cache  nextValidatorId and write in last
            unchecked {
                ++i;
            }
        }
```
## PermissionLessNodeRegistry.sol.getAllActiveValidators() : state variable *nextValidatorId* should be cached into stack varaible insted of re-reding from storage 2 times (saves  100 Gas of  Gwarmaccess )
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L501](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L501)
```solidity
501:    endIndex = endIndex > nextValidatorId ? nextValidatorId : endIndex; //@audit-gas cache  nextValidatorId
```

# [G-04]  Make instance into stack variable instead of re-creating every time calling external contract function (save 30 gas each time )

In most files this pattern is not followed, this should be followed.

