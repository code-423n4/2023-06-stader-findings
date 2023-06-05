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

2.calculateMEVTHeftPenality just need a length return from rate oracle
```solidity
    /// @inheritdoc IPenalty
    function calculateMEVTheftPenalty(bytes32 _pubkeyRoot) public override returns (uint256) {
        // Retrieve the epochs in which the validator violated the fee recipient change rule.
        uint256[] memory violatedEpochs = IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot);

        // each strike attracts `mevTheftPenaltyPerStrike` penalty
        return violatedEpochs.length * mevTheftPenaltyPerStrike;
    }
```
## Recommendation
add a function in rateOracle that only returns the length of the violatedEpochs.
