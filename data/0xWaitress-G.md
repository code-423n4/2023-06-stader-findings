1. onlyExistingPoolId iterates the entire list to find the existingPool which is very gas intensive. Use openzeppelin EnumerableSet to reduce gas
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
