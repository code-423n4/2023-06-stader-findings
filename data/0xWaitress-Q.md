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

6. extractNonBidSD should use safeTransfer instead of transfer 
```solidity
    function extractNonBidSD(uint256 lotId) external override {
        LotItem storage lotItem = lots[lotId];
        if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
        if (lotItem.highestBidAmount > 0) revert LotWasAuctioned();
        if (lotItem.sdAmount == 0) revert AlreadyClaimed();

        uint256 _sdAmount = lotItem.sdAmount;
        lotItem.sdAmount = 0;
        if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {
            revert SDTransferFailed();
        }
        emit UnsuccessfulSDAuctionExtracted(lotId, _sdAmount, staderConfig.getStaderTreasury());
    }
```

7. OperatorRewardsController.claim of 0 should either be prohibited or short-circuited from sendValue
```solidity
    function claim() external whenNotPaused {
        address operator = msg.sender;
        uint256 amount = balances[operator];
        balances[operator] -= amount;

        address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig);
        UtilLib.sendValue(operatorRewardsAddr, amount);
        emit Claimed(operatorRewardsAddr, amount);
    }
```

8. lack of function to remove node operator from permissionList on PermissionedNodeRegistry

``solidity
    // mapping of whitelisted permissioned node operator
    mapping(address => bool) public override permissionList;
```