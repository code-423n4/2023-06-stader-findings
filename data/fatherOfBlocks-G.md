**contracts/OperatorRewardsCollector.sol**
- L49 - Instead of doing a subtraction, you can directly do "balances[operator] = 0;"

**contracts/PermissionedNodeRegistry.sol**
- L224 - The operation _numValidators - totalValidatorToDeposit could be unchecked since in Line 222 the prevalidation is already performed.

**contracts/PermissionlessPool.sol**
- L279/280/281/282/283/284/285/286 - En vez de ir definiendo cada valor del length, se puede directamente iterar con un for:
for (int i = 0; i < ret.length; i++){
            ret[i] = bytesValue[ret.length - i - 1];
        }

**contracts/StaderOracle.sol**
- L603 - Divide first and then multiply by the same value: (block.number / updateFrequency) * updateFrequency. This is necessary since they are inverse operations, therefore it is a necessary expense of gas.
