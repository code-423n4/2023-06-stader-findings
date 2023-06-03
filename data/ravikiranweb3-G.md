a) There are a number of occasions where the variables are read into storage even in the cases where the data is never modified. In such cases, it is more gas efficient to read them into Memory for temporary computation.

This is one such example, but there are many such occassion where storage could be replaced with memory for optimization. Every time, the data is read only and there is no update on the state variables, it is better to read them in memory.

example, in the below function, 
-> PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];

The above reading of poolThreshold could have been in memory instead of storage.

-> PoolThresholdInfo memory poolThreshold = poolThresholdbyPoolId[_poolId];
```
function slashValidatorSD(uint256 _validatorId, uint8 _poolId) external override nonReentrant {
        address operator = UtilLib.getOperatorForValidSender(_poolId, _validatorId, msg.sender, staderConfig);
        isPoolThresholdValid(_poolId);
        PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];
        uint256 sdToSlash = convertETHToSD(poolThreshold.minThreshold);
        slashSD(operator, sdToSlash);
    }
```