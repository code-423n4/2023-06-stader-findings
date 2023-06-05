1. PoolIdArray in PoolUtils only allows a max of uint8 (256 pools)

```solidity
uint8[] public override poolIdArray;
```

2. getPoolIdArray gets the entire list which is sub-optimal. Provide access each element instead.
```solidity
    function getPoolIdArray() external view override returns (uint8[] memory) {
        return poolIdArray;
    }
```

3. onlyValidKeys can take 1 bytes and slice accordingly instead of 3 bytes.
```solidity
function onlyValidKeys(
        bytes calldata _pubkey,
        bytes calldata _preDepositSignature,
        bytes calldata _depositSignature
    )
```

4. calculation of operatorShare can be skipped if collateralETH is 0 (for permissionedPool)
```solidity
        uint256 TOTAL_STAKED_ETH = staderConfig.getStakedEthPerNode();
        uint256 collateralETH = getCollateralETH(_poolId);
        uint256 usersETH = TOTAL_STAKED_ETH - collateralETH;
        uint256 protocolFeeBps = getProtocolFee(_poolId);
        uint256 operatorFeeBps = getOperatorFee(_poolId);

        uint256 _userShareBeforeCommision = (_totalRewards * usersETH) / TOTAL_STAKED_ETH;

        protocolShare = (protocolFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();
       
>>>>>can be skipped if collateralETH is 0        operatorShare = (_totalRewards * collateralETH) / TOTAL_STAKED_ETH;
        operatorShare += (operatorFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();

        userShare = _totalRewards - protocolShare - operatorShare;
```


