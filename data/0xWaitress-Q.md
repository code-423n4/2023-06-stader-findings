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

5. onlyExistingPoolId iterates the list to find the existingPool which is very gas intensive. Use openzeppelin EnumerableSet to reduce gas
```solidity
    function getQueuedValidatorCountByPool(uint8 _poolId)
        external
        view
        override
        onlyExistingPoolId(_poolId)
        returns (uint256)
    {
        address nodeRegistry = getNodeRegistry(_poolId);
        return INodeRegistry(nodeRegistry).getTotalQueuedValidatorCount();
    }

    /// @inheritdoc IPoolUtils
    function getActiveValidatorCountByPool(uint8 _poolId)
        public
        view
        override
        onlyExistingPoolId(_poolId)
        returns (uint256)
    {
        address nodeRegistry = getNodeRegistry(_poolId);
        return INodeRegistry(nodeRegistry).getTotalActiveValidatorCount();
    }

    /// @inheritdoc IPoolUtils
    function getSocializingPoolAddress(uint8 _poolId)
        public
        view
        override
        onlyExistingPoolId(_poolId)
        returns (address)
    {
        return IStaderPoolBase(poolAddressById[_poolId]).getSocializingPoolAddress();
    }
```
