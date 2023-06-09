# [G-01] Memory variables can be cached to stack if reading and writing happening multiple times, saves almost 100 gas on each iteration 

Memory variables should be cached to stack variables if re reading happening of same variables more than once, here for each iteration it will save 100 gas almost.
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L210](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L210)
```solidity
File : contracts/PermissionedNodeRegistry.sol

                //@audit-issue gas optimization if used stack over memory variable for each iteration
               
       
                remainingOperatorCapacity[i] = getOperatorQueuedValidatorCount(i);
                selectedOperatorCapacity[i] = Math.min(remainingOperatorCapacity[i], validatorPerOperator);
                //@audit gas
                totalValidatorToDeposit += selectedOperatorCapacity[i];
                //@audit gas
                remainingOperatorCapacity[i] -= selectedOperatorCapacity[i]; //for each opertaror loop is running
                unchecked {
                    ++i;
                }
        
```
Converting above to below code save more than 100 gas for each loop iteration
Take 2 variables for caching from memory
```solidity
                uint256 a;
                uint256 b;
                
                a = remainingOperatorCapacity[i];
                b = selectedOperatorCapacity[i];
               
                a = getOperatorQueuedValidatorCount(i);
                b = Math.min(a, validatorPerOperator);
                //@audit gas,converting <x>+=<y> -> <x>=<x>+<y> costs less gas
                totalValidatorToDeposit = totalValidatorToDeposit + b;
                //@audit gas,converting <x>+=<y> -> <x>=<x>+<y> costs less gas
                a = a-b; 
                unchecked {
                    ++i;
                }
                remainingOperatorCapacity[i] = a;
                selectedOperatorCapacity[i] = b;
```
