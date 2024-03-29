
1. calculation of operatorShare can be skipped if collateralETH is 0 (for permissionedPool)
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

2. onlyExistingPoolId iterates the list to find the existingPool which is very gas intensive. Use openzeppelin EnumerableSet to reduce gas
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

3. extractNonBidSD should use safeTransfer instead of transfer for consistency
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

4. OperatorRewardsController.claim of 0 should either be prohibited or short-circuited from sendValue to avoid unnecessary call.
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

5. lack of function to remove node operator from permissionList on PermissionedNodeRegistry

``solidity
    // mapping of whitelisted permissioned node operator
    mapping(address => bool) public override permissionList;
```

7. user can only specify a full ETH as expectedAmount, but sometimes they might want to specify a decimal amount but now is not working since minEThRequiredToFinalizeRequest is dividing by 10 ** 18
```solidity
            UserWithdrawInfo memory userWithdrawInfo = userWithdrawRequests[requestId];
            uint256 requiredEth = userWithdrawInfo.ethExpected;
            uint256 lockedEthX = userWithdrawInfo.ethXAmount;
            uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);
```

8. add a way to sunset the allowance of a auctionContract on SD Collateral by setting its allowance to 0. Right now the maxApproveSD function would set allowance for auctionContract to `type(uint256).max`. But if a new auctionContract is updated, the allowance of the old auctionContract cannot be revoked.

```solidity
    /// @notice for max approval to auction contract for spending SD tokens
    function maxApproveSD() external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        address auctionContract = staderConfig.getAuctionContract();
        UtilLib.checkNonZeroAddress(auctionContract);
        IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);
    }
```

9. enableSafeMode and disableSafeMode in StaderOracle does not check if the safeMode is already enabled or disabled. This is different from all other functions which has this input validation.

```solidity
    function enableSafeMode() external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        safeMode = true;
        emit SafeModeEnabled();
    }

    function disableSafeMode() external override onlyRole(DEFAULT_ADMIN_ROLE) {
        safeMode = false;
        emit SafeModeDisabled();
    }
```
## Recommendation
add a status check before setting the safeMode to the designed value.