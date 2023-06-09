# Gas Report

## G-01 All the variables setted in the `StaderConfig` as `constants` should be raw constants declared in global file

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L89-L94

Use a file with global constants instead of calling `StaderConfig` for `constants` 
```
uint256 constant ETH_PER_NODE = 32 ether;
uint256 constant PRE_DEPOSIT_SIZE = 1 ether;
uint256 constant FULL_DEPOSIT_SIZE = 31 ether;
uint256 constant TOTAL_FEE = 100_00;
uint256 constant DECIMALS = 1e18;
uint256 constant OPERATOR_MAX_NAME_LENGTH = 255;
```

## G-02 `StaderConfig` should be an `immutable`
There is no need to have a upgrade config function, in case you need to change something inside the `StaderConfig` you can always upgrade the proxy

## G-03 Cache external calls when possible
Example [UtilLib.sol/#L123-L124](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol/#L123-L124)
Instead of;
```
uint8 poolId = IPoolUtils(_staderConfig.getPoolUtils()).getOperatorPoolId(_operator);
address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(poolId);
``` 
Use this;
```
IPoolUtils poolUtils = IPoolUtils(_staderConfig.getPoolUtils());
uint8 poolId = poolUtils.getOperatorPoolId(_operator);
address nodeRegistry = poolUtils.getNodeRegistry(poolId);
```
Similar issue on
- https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol/#L148-L149
- https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L199-L200

## G-04 Remove unnecesary `nonReentrant` modifiers
In [Penalty.sol#L98](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L98) there is no reentrancy issue, you could safely remove this modifier
In [Auction.sol/#L93](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol/#L93), [Auction.sol/#L120](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol/#L120) it following correctly the Check Effects iterations pattern, so its safe to remove the nonReentrant guard


## G-05 External calls inside loop for constant data should be cached
On [UserWithdrawalManager.sol/#L139](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol/#L139) the external call `staderConfig.getMinBlockDelayToFinalizeWithdrawRequest()` should be caches to avoid multiple external calls that will return the same value.

## G-06 Avoid read multiple time the storage
Example on [UserWithdrawalManager.sol/#L109-L110](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol/#L109-L110)
Instead of
```
nextRequestId++;
return nextRequestId - 1;
```
Use
```
return nextRequestId++;
```

## G-07 Use named return is cheaper to declarate an extra variable
Example on [VaultFactory.sol/#L48-L60](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol/#L48-L60) this pattern is cheaper:
```diff
diff --git a/contracts/factory/VaultFactory.sol b/contracts/factory/VaultFactory.sol
index 3b8bb1d..ae58ab3 100644
--- a/contracts/factory/VaultFactory.sol
+++ b/contracts/factory/VaultFactory.sol
@@ -36,27 +36,25 @@ contract VaultFactory is IVaultFactory, Initializable, AccessControlUpgradeable
         uint256 _operatorId,
         uint256 _validatorCount,
         uint256 _validatorId
-    ) public override onlyRole(NODE_REGISTRY_CONTRACT) returns (address) {
+    ) public override onlyRole(NODE_REGISTRY_CONTRACT) returns (address withdrawVaultAddress) {
         bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));
-        address withdrawVaultAddress = ClonesUpgradeable.cloneDeterministic(vaultProxyImplementation, salt);
+        withdrawVaultAddress = ClonesUpgradeable.cloneDeterministic(vaultProxyImplementation, salt);
         VaultProxy(payable(withdrawVaultAddress)).initialise(true, _poolId, _validatorId, address(staderConfig));
 
         emit WithdrawVaultCreated(withdrawVaultAddress);
-        return withdrawVaultAddress;
     }
 
     function deployNodeELRewardVault(uint8 _poolId, uint256 _operatorId)
         public
         override
         onlyRole(NODE_REGISTRY_CONTRACT)
-        returns (address)
+        returns (address nodeELRewardVaultAddress)
     {
         bytes32 salt = sha256(abi.encode(_poolId, _operatorId));
-        address nodeELRewardVaultAddress = ClonesUpgradeable.cloneDeterministic(vaultProxyImplementation, salt);
+        nodeELRewardVaultAddress = ClonesUpgradeable.cloneDeterministic(vaultProxyImplementation, salt);
         VaultProxy(payable(nodeELRewardVaultAddress)).initialise(false, _poolId, _operatorId, address(staderConfig));
 
         emit NodeELRewardVaultCreated(nodeELRewardVaultAddress);
-        return nodeELRewardVaultAddress;
     }
```

## G-08 Tight packing struct `UserWithdrawInfo`
Plase read [this article](https://fravoll.github.io/solidity-patterns/tight_variable_packing.html)

Struct `UserWithdrawInfo` on (UserWithdrawalManager.sol#L43-L46
)[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L43-L46]
can be packed from:
```
    struct UserWithdrawInfo {
        address payable owner; // address that can claim eth on behalf of this request
        uint256 ethXAmount; //amount of ethX share locked for withdrawal
        uint256 ethExpected; //eth requested according to given share and exchangeRate
        uint256 ethFinalized; // final eth for claiming according to finalize exchange rate
        uint256 requestBlock; // block number of withdraw request
    }
```

To:
```
    struct UserWithdrawInfo {
        address payable owner; // address that can claim eth on behalf of this request
        // safe to asume that user wont lock more than max(uint128) ether
        uint128 ethXAmount; //amount of ethX share locked for withdrawal
        // safe to asume that user wont expect more than max(uint128) ether
        uint128 ethExpected; //eth requested according to given share and exchangeRate
        // safe to asume that user wont claim more than max(uint128) ether
        uint128 ethFinalized; // final eth for claiming according to finalize exchange rate
        // safe to asume that block is lower than max(uint128) ether
        uint128 requestBlock; // block number of withdraw request
    }
```




## G-09 `X = X + Y` is cheaper than `X += Y` with local variables
Example in [UserWithdrawalManager.sol/#L145-L147](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol/#L145-L147)
Instead of
```
ethRequestedForWithdraw -= requiredEth;
lockedEthXToBurn += lockedEthX;
ethToSendToFinalizeRequest += minEThRequiredToFinalizeRequest;
```
Use
```
ethRequestedForWithdraw = ethRequestedForWithdraw - requiredEth;
lockedEthXToBurn = lockedEthXToBurn + lockedEthX;
ethToSendToFinalizeRequest = ethToSendToFinalizeRequest + minEThRequiredToFinalizeRequest;
```

## G-10 `X = X * Y` is cheaper than `X *= Y` with local variables
Example [SDCollateral.sol#L175-L176](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L175-L176)