**contracts/SocializingPool.sol**
- L9 - A interface is imported that is not used in any way within the IPermissionlessNodeRegistry interface, this generate that when the interface is deployed in a blockchain and in an explorer the entire code is seen, it becomes difficult to understand the code.
In addition to the extra cost of gas that is generated.

- L145/146/157 - In the _claim() function a for loop is used to iterate the _index array, but inside is used to iterate multiple arrays. This is a problem because it is not checked that they have the same length. This is necessary as it would either revert without knowing the reason or execute without using all the values ​​within the array of: _amountSD, _amountETH and _merkleProof.

- L174 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/StaderStakePoolsManager.sol**
- L276 - Before doing a division by rounding, it should be checked that it is different from zero, since otherwise it would generate an exception without saying the reason, since it is an input of the function without any limitation.


**contracts/PoolSelector.sol**
- L92/93 - Before doing a division by poolDepositSize, it should be checked that it is different from zero, otherwise it would generate an exception without saying the reason.


**contracts/PoolUtils.sol**
- L263/266 - Before making a division by staderConfig.getTotalFee(), it should be checked that it is different from zero, since otherwise it would generate an exception without saying the reason.


**contracts/SDCollateral.sol**
- L211/212 - Before doing a division by sdPriceInETH, you should check that it is different from zero, otherwise it would generate an exception without saying the reason.


**contracts/PermissionlessPool.sol**
- L5/12 - A interface and a library are imported that are not used in any way within the ValidatorStatus library and IStaderStakePoolManager interface, these generates that when the interface and library are deployed in a blockchain and in an explorer the entire code is seen, it becomes difficult to understand the code.
In addition to the extra cost of gas that is generated.

- L225/227/228/229/234/235/236 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as sha256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/PermissionedNodeRegistry.sol**
- L140/146/155/156 - It is not checked that the three arrays have the same length as keyCount, this is important because the three arrays are iterated within the for: _pubkey, _preDepositSignature, _depositSignature.


**contracts/NodeELRewardVault.sol**
- L8 - The INodeRegistry interface is imported but never used, so it should be removed.
