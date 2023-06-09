### [Low-01] `burnFrom()` In `ETHx.sol` contract file gives `BURNER_ROLE` to burn any accounts Fund
```solidity
 function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {
        _burn(account, amount); // @audit have power to burn any contracts fund without mentioning them
    }
```
*Instances(1)*
```
File: contracts/ETHx.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L57
```

### [Low-02] Some Functions should be `payable` which interacting with `ETH` like transfering `ETH` to other contracts.
```solidity
- function withdraw() external override {

+ function withdraw() external override payable{
```

*Instances(5)*
```
File: contracts/NodeELRewardVault.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L24
```
```
File: contracts/ValidatorWithdrawalVault.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L30
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L54
```
```
File: contracts/StaderInsuranceFund.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L41
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L60
```

### [Low-03] `ETH` could be locked inside a contract
Contract implement payable function like `depositFor()`, posibility any user can sent `eth` by mistakely, and as there is no rescue function present in contract, `eth` will remain locked in that contract

*Similarly Another case*

```solidity
    fallback(bytes calldata _input) external payable returns (bytes memory) { 
        address vaultImplementation = isValidatorWithdrawalVault 
            ? staderConfig.getValidatorWithdrawalVaultImplementation()
            : staderConfig.getNodeELRewardVaultImplementation(); 
        (bool success, bytes memory data) = vaultImplementation.delegatecall(_input); 
        if (!success) {
            revert(string(data));
        }
        return data;
    }
```

*Similarly Another case*

```solidity
    receive() external payable { 
        emit ETHReceived(msg.sender, msg.value);
    }
```

*Instances(3)*
```
File: contracts/VaultProxy.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L41-L50
```
```
File: contracts/OperatorRewardsCollector.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L40
```
```
File: contracts/ValidatorWithdrawalVault.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L26-L28
```

### [Low-04] `depositFor()` even work when contract is in `paused` mode
There should be `whenNotPaused` modifier present
```solidity
-    function depositFor(address _receiver) external payable { 

+    function depositFor(address _receiver) external payable whenNotPaused { 
        balances[_receiver] += msg.value;

        emit DepositedFor(msg.sender, _receiver, msg.value); // @audit ETH can locked here
    }
```

*Instances(1)*
```
File: contracts/OperatorRewardsCollector.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L40-L45
```

### [Low-05] Should Check for `Zero address` before sending `Eth`
Here `UtilLib.getOperatorRewardAddress` return `receiver` address,
which getting that via `INodeRegistry(nodeRegistry).getOperatorRewardAddress(operatorId)`

so this could be `0x0000..` address. So should validate  `operatorRewardsAddr` before calling `UtilLib.sendValue(`
```solidity
       .............
       .............

        address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig); 
        UtilLib.sendValue(operatorRewardsAddr, amount);
        emit Claimed(operatorRewardsAddr, amount);
```

*Instances(1)*
```
File: contracts/OperatorRewardsCollector.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L51
```

### [Low-06] Admin can change critical parameter like `_staerConfig` while `contract` in `paused` mode
Adin should not able to do that, at least use some timelock like functionality.
```solidity
    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) { 
        UtilLib.checkNonZeroAddress(_staderConfig);
        staderConfig = IStaderConfig(_staderConfig);
        emit UpdatedStaderConfig(_staderConfig);
```

*Instances(1)*
```
File: contracts/OperatorRewardsCollector.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L56-L60
```

### [Low-07] `addNewPool()` Function could be used to create `pool` with a arbitary `_poolId`, which could be problematic in upcoming future.
In my opinion `_poolId` should cached in stack and which could incremented gradually, which could be easily trackable

```solidity
    function addNewPool(uint8 _poolId, address _poolAddress) external override { 
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        UtilLib.checkNonZeroAddress(_poolAddress);
        verifyNewPool(_poolId, _poolAddress);
        poolIdArray.push(_poolId);
        poolAddressById[_poolId] = _poolAddress;
        emit PoolAdded(_poolId, _poolAddress);
    }
```

*Instances(1)*
```
File: contracts/PoolUtils.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L40-L47
```

### [Low-08] While contract receive `eth` via `fallback` should `emit` something

```solidity
    fallback(bytes calldata _input) external payable returns (bytes memory) { 
        address vaultImplementation = isValidatorWithdrawalVault 
            ? staderConfig.getValidatorWithdrawalVaultImplementation()
            : staderConfig.getNodeELRewardVaultImplementation(); 
        (bool success, bytes memory data) = vaultImplementation.delegatecall(_input); 
        if (!success) {
            revert(string(data));
        }
        return data;
    }
```
*Instances(1)*
```
File: contracts/VaultProxy.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L41-L50
```

### [Low-09] Should Check address before making low level to a `contract`
Here both `staderConfig.getValidatorWithdrawalVaultImplementation()` and `staderConfig.getNodeELRewardVaultImplementation()` can return `0x00.....`, So before making any low level call to these returned address `function` should ensure that those are not zero address, Otherwise call get successful without revert. 
```solidity
    fallback(bytes calldata _input) external payable returns (bytes memory) { 
        address vaultImplementation = isValidatorWithdrawalVault 
            ? staderConfig.getValidatorWithdrawalVaultImplementation()
            : staderConfig.getNodeELRewardVaultImplementation(); 
        (bool success, bytes memory data) = vaultImplementation.delegatecall(_input); 
        if (!success) {
            revert(string(data));
        }
        return data;
    }
```

*Instances(1)*
```
File: contracts/VaultProxy.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L41-L50
```


### [Low-10] `nodeRegistry` not properly validated here, before making further call to that

```solidity
   function getUpdatedPenaltyAmount(
        uint8 _poolId,
        uint256 _validatorId,
        IStaderConfig _staderConfig
    ) internal returns (uint256) {
        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId); // @audit L::  Not properly validate here
        (, bytes memory pubkey, , , , , , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);
        bytes[] memory pubkeyArray = new bytes[](1);
        pubkeyArray[0] = pubkey;
        IPenalty(_staderConfig.getPenaltyContract()).updateTotalPenaltyAmount(pubkeyArray);
        return IPenalty(_staderConfig.getPenaltyContract()).totalPenaltyAmount(pubkey);
    }
```

*Instances(1)*
```
File: contracts/ValidatorWithdrawalVault.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L144-L149
```
