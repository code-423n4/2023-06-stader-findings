https://github.com/code-423n4/2023-06-stader/blob/19d6283a4e21ec928d6fd22ee0e42410c05ef3e5/contracts/PermissionedNodeRegistry.sol#L532 

```solidity 
function getOperatorTotalNonTerminalKeys(
        address _nodeOperator,
        uint256 _startIndex,
        uint256 _endIndex
    ) {
  ...
  int256 validatorCount = getOperatorTotalKeys(operatorId); 
   _endIndex = _endIndex > validatorCount ? validatorCount : _endIndex;  //
  ...
}
```

line 532  `_endIndex=`   logic is very weird, because  _endIndex always=validatorCount ,  In this function `int256 validatorCount = getOperatorTotalKeys(operatorId);` , but in line 702 https://github.com/code-423n4/2023-06-stader/blob/19d6283a4e21ec928d6fd22ee0e42410c05ef3e5/contracts/PermissionedNodeRegistry.sol#L702,  where `getOperatorTotalNonTerminalKeys` function is called, we can see that `totalKeys = getOperatorTotalKeys(_operatorId);` ,and totalKeys is _endIndex ,so  ternary expressions is useless


