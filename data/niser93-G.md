
# Gas Optimizations
|       |Issue  | Instances |
|:-----:|:-----:|:---------:|
|GO-01|Multiplication/division by 2 should use bit shifting|     2     |
|GO-02|Avoid to define variable which are used once|    28     |
|GO-03|Avoid useless checks|    1     |


## Optimization details
| Test                           | Name                                                                                 | Gas     |
|--------------------------------|--------------------------------------------------------------------------------------|---------|
| AuctionTest                    | test_JustToIncreaseCoverage()                                                        | -2096   |
| AuctionTest                    | test_extractNonBidSD(uint256,uint64)                                                 | 2       |
| NodeELRewardVaultTest          | test_withdraw(uint128)                                                               | -2739   |
| PenaltyTest                    | test_JustToIncreaseCoverage()                                                        | -6825   |
| PenaltyTest                    | test_additionalPenaltyAmount(address,uint256,bytes)                                  | 6       |
| PenaltyTest                    | test_calculateMEVTheftPenalty()                                                      | -38     |
| PenaltyTest                    | test_updateTotalPenaltyAmount()                                                      | -94     |
| PermissionedNodeRegistryTest   | test_ActivateNodeOperator(uint64,uint64)                                             | -6      |
| PermissionedNodeRegistryTest   | test_AllocateValidatorsAndUpdateOperatorId()                                         | -12     |
| PermissionedNodeRegistryTest   | test_DeactivateNodeOperator(uint64,uint64)                                           | -6      |
| PermissionedNodeRegistryTest   | test_JustToIncreaseCoverage()                                                        | -200    |
| PermissionedNodeRegistryTest   | test_OnboardOperator(string,uint64,uint64)                                           | -6      |
| PermissionedNodeRegistryTest   | test_addValidatorKeys()                                                              | -6      |
| PermissionedNodeRegistryTest   | test_addValidatorKeysOPCrossingMaxNonTerminalKeys()                                  | -6      |
| PermissionedNodeRegistryTest   | test_addValidatorKeysWithInsufficientSDCollateral()                                  | -6      |
| PermissionedNodeRegistryTest   | test_addValidatorKeysWithInvalidKeyCount()                                           | -6      |
| PermissionedNodeRegistryTest   | test_addValidatorKeysWithMisMatchingInputs()                                         | -6      |
| PermissionedNodeRegistryTest   | test_getAllActiveValidators(uint256,uint256)                                         | -10110  |
| PermissionedNodeRegistryTest   | test_getAllActiveValidatorsWithZeroPageNumber(uint256)                               | -6      |
| PermissionedNodeRegistryTest   | test_getOperatorTotalNonTerminalKeys(uint64,uint64,uint256,uint256)                  | -6      |
| PermissionedNodeRegistryTest   | test_getOperatorTotalNonTerminalKeysInvalidPagination(uint64,uint64,uint256,uint256) | -6      |
| PermissionedNodeRegistryTest   | test_getValidatorsByOperator()                                                       | -6      |
| PermissionedNodeRegistryTest   | test_markReadyToDepositValidator()                                                   | -6      |
| PermissionedNodeRegistryTest   | test_updateNextQueuedValidatorIndex(uint64,uint256)                                  | -6      |
| PermissionedNodeRegistryTest   | test_updateOperatorDetailWithInvalidName(string,uint64,uint64,uint64)                | -6      |
| PermissionedNodeRegistryTest   | test_updateOperatorDetailWithZeroRewardAddr(string,uint64)                           | -6      |
| PermissionedNodeRegistryTest   | test_updateOperatorDetails(string,uint64,uint64,uint64)                              | -6      |
| PermissionedNodeRegistryTest   | test_withdrawnValidators()                                                           | -6      |
| PermissionlessNodeRegistryTest | test_JustToIncreaseCoverage()                                                        | -111904 |
| PermissionlessNodeRegistryTest | test_OnboardOperatorWhenPaused(string,uint64,uint64)                                 | -1      |
| PermissionlessNodeRegistryTest | test_changeSocializingPoolStateWithSomeCoolDown(string,uint64,uint64)                | -2      |
| PermissionlessNodeRegistryTest | test_getAllActiveValidators(uint256,uint256)                                         | -11875  |
| PermissionlessNodeRegistryTest | test_getAllActiveValidatorsWithZeroPageNumber(uint256)                               | -31     |
| PermissionlessNodeRegistryTest | test_getNodeELVaultAddressForOptOutOperators(uint256,uint256)                        | 59      |
| PermissionlessNodeRegistryTest | test_markReadyToDepositValidator()                                                   | -124    |
| PermissionlessNodeRegistryTest | test_withdrawnValidators()                                                           | -15472  |
| SDCollateralTest               | testFail_depositSDAsCollateral_withInsufficientApproval(uint256,uint256)             | -7      |
| SDCollateralTest               | test_JustToIncreaseCoverage()                                                        | -26458  |
| SDCollateralTest               | test_depositSDAsCollateral(uint128,uint128,uint16)                                   | -40     |
| SDCollateralTest               | test_getRemainingSDToBond(uint256,uint256)                                           | -152    |
| SDCollateralTest               | test_getRewardEligibleSD(uint256)                                                    | -176    |
| SDCollateralTest               | test_sd_eth_converters(uint128)                                                      | -65     |
| SDCollateralTest               | test_slashValidatorSD()                                                              | -222    |
| SDCollateralTest               | test_slashValidatorSD_auctionLotNotCreated_whenNoCollateral()                        | -49     |
| SDCollateralTest               | test_withdraw(uint128,uint128)                                                       | -161    |
| SDCollateralTest               | test_withdraw_revertIfPoolThresholdNotSet(uint256)                                   | 6       |
| SDCollateralTest               | test_withdraw_reverts_InsufficientSDToWithdraw(uint128,uint128)                      | -102    |
| --                             | --                                                                                   |         |
| Total                          |                                                                                      | -188990 |


## GO-01
### Multiplication/division by 2 should use bit shifting
#### Description
```<x> * 2``` is equivalent to ```<x> << 1``` and ```<x> / 2``` is the same as ```<x> >> 1```. The ```MUL``` and ```DIV``` opcodes cost 5 gas, whereas ```SHL``` and ```SHR``` only cost 3 gas.



```diff
- n*2
+ n << 1
```            

```diff
- n/2
+ n >> 1
```            

There are 2 instances of this issue not reported in bot race:

```diff
File: Auction.sol

  
- 38         duration = 2 * MIN_AUCTION_DURATION;
+ 38         duration = MIN_AUCTION_DURATION<<1;

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L38](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L38)

```diff
File: StaderOracle.sol

  
-290         if ((submissionCount == (2 * trustedNodesCount) / 3 + 1)) {
+290         if ((submissionCount == (trustedNodesCount<<1) / 3 + 1)) {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L148](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L148)

            

#### Gas Report:
```
Using forge snapshot
```

```diff
|   Test Name                    | Gas     |
|--------------------------------|---------|
-| test_JustToIncreaseCoverage() | 2598490 |
+| test_JustToIncreaseCoverage() | 2596394 |
|--------------------------------|---------|
|                                |   -2096 |

The saved gas is 2096.
```
                

```diff
|   Test Name                           | Gas    |
|---------------------------------------|--------|
-| test_extractNonBidSD(uint256,uint64) | 203347 |
+| test_extractNonBidSD(uint256,uint64) | 203349 |
|---------------------------------------|--------|
|                                       |      2 |

The average lost gas is 2.
```
                

| Test        | Name                                 | Gas   |
|-------------|--------------------------------------|-------|
| AuctionTest | test_JustToIncreaseCoverage()        | -2096 |
| AuctionTest | test_extractNonBidSD(uint256,uint64) | 2     |
| --          | --                                   |       |
| Total       |                                      | -2094 |


## GO-02
### Avoid to define variable which are used once
#### Description

Avoid to declare variable if you use it once.
Sometimes it could get worse in human readability, so we report only cases where it is reasonable.

There are other instances whose we don't report, because there are some contract whose aren't covered by test.

There are 3 instances of this issue:

```diff
File: NodeELRewardVault.sol

    function withdraw() external override {
        uint8 poolId = IVaultProxy(address(this)).poolId();
-       uint256 operatorId = IVaultProxy(address(this)).id();
        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
        uint256 totalRewards = address(this).balance;
        if (totalRewards == 0) {
            revert NotEnoughRewardToWithdraw();
        }
        (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
            .calculateRewardShare(poolId, totalRewards);

        // Distribute rewards
        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{value: userShare}();
        // slither-disable-next-line arbitrary-send-eth
        UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
-       address operator = UtilLib.getOperatorAddressByOperatorId(poolId, operatorId, staderConfig);
        IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
-           operator
+           UtilLib.getOperatorAddressByOperatorId(poolId, IVaultProxy(address(this)).id(), staderConfig);
        );

        emit Withdrawal(protocolShare, operatorShare, userShare);
    }

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L24-L45](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L24-L45)


```diff
File: Penalty.sol

[L54-L55]
- bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey);
- additionalPenaltyAmount[pubkeyRoot] += _amount;
+ additionalPenaltyAmount[UtilLib.getPubkeyRoot(_pubkey)] += _amount;

[L106-L111]
- uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
- uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);

  // Compute the total penalty for the given validator public key,
  // taking into account additional penalties and penalty reversals from the DAO.
- uint256 totalPenalty = _mevTheftPenalty + _missedAttestationPenalty + additionalPenaltyAmount[pubkeyRoot];
+ uint256 totalPenalty = calculateMEVTheftPenalty(pubkeyRoot) + calculateMissedAttestationPenalty(pubkeyRoot) + additionalPenaltyAmount[pubkeyRoot];

[L123-L129]
function calculateMEVTheftPenalty(bytes32 _pubkeyRoot) public override returns (uint256) {
    // Retrieve the epochs in which the validator violated the fee recipient change rule.
-   uint256[] memory violatedEpochs = IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot);

    // each strike attracts `mevTheftPenaltyPerStrike` penalty
-   return violatedEpochs.length * mevTheftPenaltyPerStrike;
+   return IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot).length * mevTheftPenaltyPerStrike;
}
```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L55-L56](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L55-L56)


```diff
File: PermissionedNodeRegistry.sol

[L106-L131]
    function onboardNodeOperator(string calldata _operatorName, address payable _operatorRewardAddress)
        external
        override
        whenNotPaused
-       returns (address feeRecipientAddress)
+       returns (address)
    {
        address poolUtils = staderConfig.getPoolUtils();
        if (IPoolUtils(poolUtils).poolAddressById(POOL_ID) != staderConfig.getPermissionedPool()) {
            revert DuplicatePoolIDOrPoolNotAdded();
        }
        IPoolUtils(poolUtils).onlyValidName(_operatorName);
        UtilLib.checkNonZeroAddress(_operatorRewardAddress);
        if (nextOperatorId > maxOperatorId) {
            revert MaxOperatorLimitReached();
        }
        if (!permissionList[msg.sender]) {
            revert NotAPermissionedNodeOperator();
        }
        //checks if operator already onboarded in any pool of protocol
        if (IPoolUtils(poolUtils).isExistingOperator(msg.sender)) {
            revert OperatorAlreadyOnBoardedInProtocol();
        }
-       feeRecipientAddress = staderConfig.getPermissionedSocializingPool();
        onboardOperator(_operatorName, _operatorRewardAddress);
-       return feeRecipientAddress;
+       return staderConfig.getPermissionedSocializingPool();
    }


[]

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L106-L131](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L106-L131)


```diff
File: PermissionlessNodeRegistry.sol

    function isActiveValidator(uint256 _validatorId) internal view returns (bool) {
-       Validator memory validator = validatorRegistry[_validatorId];
-       return validator.status == ValidatorStatus.DEPOSITED;
+       return validatorRegistry[_validatorId].status == ValidatorStatus.DEPOSITED;
    }

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L701-L704](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L701-L704)


```diff
File: SDCollateral.sol

[L43-L52]
    function depositSDAsCollateral(uint256 _sdAmount) external override {
-       address operator = msg.sender;
-       operatorSDBalance[operator] += _sdAmount;
+       operatorSDBalance[msg.sender] += _sdAmount;

-       if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {
+       if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }

-       emit SDDeposited(operator, _sdAmount);
+       emit SDDeposited(msg.sender, _sdAmount);
    }


[L58-L73]
    function withdraw(uint256 _requestedSD) external override {
-       address operator = msg.sender;
-       uint256 opSDBalance = operatorSDBalance[operator];
+       uint256 opSDBalance = operatorSDBalance[msg.sender];

-       if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
+       if (opSDBalance < getOperatorWithdrawThreshold(msg.sender) + _requestedSD) {
            revert InsufficientSDToWithdraw(opSDBalance);
        }
-       operatorSDBalance[operator] -= _requestedSD;
+       operatorSDBalance[msg.sender] -= _requestedSD;

        // cannot use safeERC20 as this contract is an upgradeable contract
-       if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
+       if (!IERC20(staderConfig.getStaderToken()).transfer(payable(msg.sender), _requestedSD)) {
            revert SDTransferFailed();
        }

-       emit SDWithdrawn(operator, _requestedSD);
+       emit SDWithdrawn(msg.sender, _requestedSD);
    }

[L78-L84]
    function slashValidatorSD(uint256 _validatorId, uint8 _poolId) external override nonReentrant {
        address operator = UtilLib.getOperatorForValidSender(_poolId, _validatorId, msg.sender, staderConfig);
        isPoolThresholdValid(_poolId);
        PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];
-       uint256 sdToSlash = convertETHToSD(poolThreshold.minThreshold);
-       slashSD(operator, sdToSlash);
+       slashSD(operator, convertETHToSD(poolThreshold.minThreshold));
    }

[L90-L99]
    function slashSD(address _operator, uint256 _sdToSlash) internal {
-       uint256 sdBalance = operatorSDBalance[_operator];
-       uint256 sdSlashed = Math.min(_sdToSlash, sdBalance);
+       uint256 sdSlashed = Math.min(_sdToSlash, operatorSDBalance[_operator]);
        if (sdSlashed == 0) {
            return;
        }
        operatorSDBalance[_operator] -= sdSlashed;
        IAuction(staderConfig.getAuctionContract()).createLot(sdSlashed);
        emit SDSlashed(_operator, staderConfig.getAuctionContract(), sdSlashed);
    }

[L166-L177]
    function getMinimumSDToBond(uint8 _poolId, uint256 _numValidator)
        public
        view
        override
        returns (uint256 _minSDToBond)
    {
        isPoolThresholdValid(_poolId);
-       PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];

-       _minSDToBond = convertETHToSD(poolThreshold.minThreshold);
+       _minSDToBond = convertETHToSD(poolThresholdbyPoolId[_poolId].minThreshold);
        _minSDToBond *= _numValidator;
    }

[L205-L213]
    function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {
-       uint256 sdPriceInETH = IStaderOracle(staderConfig.getStaderOracle()).getSDPriceInETH();
-       return (_sdAmount * sdPriceInETH) / staderConfig.getDecimals();
+       return (_sdAmount * IStaderOracle(staderConfig.getStaderOracle()).getSDPriceInETH()) / staderConfig.getDecimals();
    }

    function convertETHToSD(uint256 _ethAmount) public view override returns (uint256) {
-       uint256 sdPriceInETH = IStaderOracle(staderConfig.getStaderOracle()).getSDPriceInETH();
-       return (_ethAmount * staderConfig.getDecimals()) / sdPriceInETH;
+       return (_ethAmount * staderConfig.getDecimals()) / IStaderOracle(staderConfig.getStaderOracle()).getSDPriceInETH();
    }
```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L43-L52](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L43-L52)


#### Gas Report:
```
Using forge snapshot
```

```diff
|   Test Name             | Gas    |
|-------------------------|--------|
-| test_withdraw(uint128) | 355541 |
+| test_withdraw(uint128) | 352802 |
|-------------------------|--------|
|                         |  -2739 |

The average saved gas is 2739.
```
                

```diff
|   Test Name                    | Gas     |
|--------------------------------|---------|
-| test_JustToIncreaseCoverage() | 2347220 |
+| test_JustToIncreaseCoverage() | 2340395 |
|--------------------------------|---------|
|                                |   -6825 |

The saved gas is 6825.
```
                

```diff
|   Test Name                                          | Gas   |
|------------------------------------------------------|-------|
-| test_additionalPenaltyAmount(address,uint256,bytes) | 61704 |
+| test_additionalPenaltyAmount(address,uint256,bytes) | 61710 |
|------------------------------------------------------|-------|
|                                                      |     6 |

The average lost gas is 6.
```
                

```diff
|   Test Name                      | Gas   |
|----------------------------------|-------|
-| test_calculateMEVTheftPenalty() | 32776 |
+| test_calculateMEVTheftPenalty() | 32738 |
|----------------------------------|-------|
|                                  |   -38 |

The saved gas is 38.
```
                

```diff
|   Test Name                      | Gas    |
|----------------------------------|--------|
-| test_updateTotalPenaltyAmount() | 227942 |
+| test_updateTotalPenaltyAmount() | 227848 |
|----------------------------------|--------|
|                                  |    -94 |

The saved gas is 94.
```
                

```diff
|   Test Name                               | Gas    |
|-------------------------------------------|--------|
-| test_ActivateNodeOperator(uint64,uint64) | 268942 |
+| test_ActivateNodeOperator(uint64,uint64) | 268936 |
|-------------------------------------------|--------|
|                                           |     -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name                                   | Gas     |
|-----------------------------------------------|---------|
-| test_AllocateValidatorsAndUpdateOperatorId() | 3526864 |
+| test_AllocateValidatorsAndUpdateOperatorId() | 3526852 |
|-----------------------------------------------|---------|
|                                               |     -12 |

The saved gas is 12.
```
                

```diff
|   Test Name                                 | Gas    |
|---------------------------------------------|--------|
-| test_DeactivateNodeOperator(uint64,uint64) | 243451 |
+| test_DeactivateNodeOperator(uint64,uint64) | 243445 |
|---------------------------------------------|--------|
|                                             |     -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name                    | Gas     |
|--------------------------------|---------|
-| test_JustToIncreaseCoverage() | 5303416 |
+| test_JustToIncreaseCoverage() | 5303216 |
|--------------------------------|---------|
|                                |    -200 |

The saved gas is 200.
```
                

```diff
|   Test Name                                 | Gas    |
|---------------------------------------------|--------|
-| test_OnboardOperator(string,uint64,uint64) | 400997 |
+| test_OnboardOperator(string,uint64,uint64) | 400991 |
|---------------------------------------------|--------|
|                                             |     -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name              | Gas     |
|--------------------------|---------|
-| test_addValidatorKeys() | 1700563 |
+| test_addValidatorKeys() | 1700557 |
|--------------------------|---------|
|                          |      -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                                          | Gas    |
|------------------------------------------------------|--------|
-| test_addValidatorKeysOPCrossingMaxNonTerminalKeys() | 246860 |
+| test_addValidatorKeysOPCrossingMaxNonTerminalKeys() | 246854 |
|------------------------------------------------------|--------|
|                                                      |     -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                                          | Gas    |
|------------------------------------------------------|--------|
-| test_addValidatorKeysWithInsufficientSDCollateral() | 242052 |
+| test_addValidatorKeysWithInsufficientSDCollateral() | 242046 |
|------------------------------------------------------|--------|
|                                                      |     -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                                 | Gas    |
|---------------------------------------------|--------|
-| test_addValidatorKeysWithInvalidKeyCount() | 225015 |
+| test_addValidatorKeysWithInvalidKeyCount() | 225009 |
|---------------------------------------------|--------|
|                                             |     -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                                   | Gas    |
|-----------------------------------------------|--------|
-| test_addValidatorKeysWithMisMatchingInputs() | 226380 |
+| test_addValidatorKeysWithMisMatchingInputs() | 226374 |
|-----------------------------------------------|--------|
|                                               |     -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                                   | Gas     |
|-----------------------------------------------|---------|
-| test_getAllActiveValidators(uint256,uint256) | 1904138 |
+| test_getAllActiveValidators(uint256,uint256) | 1894028 |
|-----------------------------------------------|---------|
|                                               |  -10110 |

The average saved gas is 10110.
```
                

```diff
|   Test Name                                             | Gas     |
|---------------------------------------------------------|---------|
-| test_getAllActiveValidatorsWithZeroPageNumber(uint256) | 1778942 |
+| test_getAllActiveValidatorsWithZeroPageNumber(uint256) | 1778936 |
|---------------------------------------------------------|---------|
|                                                         |      -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name                                                          | Gas     |
|----------------------------------------------------------------------|---------|
-| test_getOperatorTotalNonTerminalKeys(uint64,uint64,uint256,uint256) | 1729560 |
+| test_getOperatorTotalNonTerminalKeys(uint64,uint64,uint256,uint256) | 1729554 |
|----------------------------------------------------------------------|---------|
|                                                                      |      -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name                                                                           | Gas     |
|---------------------------------------------------------------------------------------|---------|
-| test_getOperatorTotalNonTerminalKeysInvalidPagination(uint64,uint64,uint256,uint256) | 1723913 |
+| test_getOperatorTotalNonTerminalKeysInvalidPagination(uint64,uint64,uint256,uint256) | 1723907 |
|---------------------------------------------------------------------------------------|---------|
|                                                                                       |      -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name                     | Gas     |
|---------------------------------|---------|
-| test_getValidatorsByOperator() | 1722374 |
+| test_getValidatorsByOperator() | 1722368 |
|---------------------------------|---------|
|                                 |      -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                         | Gas     |
|-------------------------------------|---------|
-| test_markReadyToDepositValidator() | 1941624 |
+| test_markReadyToDepositValidator() | 1941618 |
|-------------------------------------|---------|
|                                     |      -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                                          | Gas    |
|------------------------------------------------------|--------|
-| test_updateNextQueuedValidatorIndex(uint64,uint256) | 285459 |
+| test_updateNextQueuedValidatorIndex(uint64,uint256) | 285453 |
|------------------------------------------------------|--------|
|                                                      |     -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name                                                            | Gas    |
|------------------------------------------------------------------------|--------|
-| test_updateOperatorDetailWithInvalidName(string,uint64,uint64,uint64) | 302797 |
+| test_updateOperatorDetailWithInvalidName(string,uint64,uint64,uint64) | 302791 |
|------------------------------------------------------------------------|--------|
|                                                                        |     -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name                                                 | Gas    |
|-------------------------------------------------------------|--------|
-| test_updateOperatorDetailWithZeroRewardAddr(string,uint64) | 301505 |
+| test_updateOperatorDetailWithZeroRewardAddr(string,uint64) | 301499 |
|-------------------------------------------------------------|--------|
|                                                             |     -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name                                              | Gas    |
|----------------------------------------------------------|--------|
-| test_updateOperatorDetails(string,uint64,uint64,uint64) | 315327 |
+| test_updateOperatorDetails(string,uint64,uint64,uint64) | 315321 |
|----------------------------------------------------------|--------|
|                                                          |     -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name                 | Gas     |
|-----------------------------|---------|
-| test_withdrawnValidators() | 1993128 |
+| test_withdrawnValidators() | 1993122 |
|-----------------------------|---------|
|                             |      -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                    | Gas     |
|--------------------------------|---------|
-| test_JustToIncreaseCoverage() | 5531119 |
+| test_JustToIncreaseCoverage() | 5421424 |
|--------------------------------|---------|
|                                | -109695 |

The saved gas is 109695.
```
                

```diff
|   Test Name                                           | Gas   |
|-------------------------------------------------------|-------|
-| test_OnboardOperatorWhenPaused(string,uint64,uint64) | 50497 |
+| test_OnboardOperatorWhenPaused(string,uint64,uint64) | 50496 |
|-------------------------------------------------------|-------|
|                                                       |    -1 |

The average saved gas is 1.
```
                

```diff
|   Test Name                                                            | Gas    |
|------------------------------------------------------------------------|--------|
-| test_changeSocializingPoolStateWithSomeCoolDown(string,uint64,uint64) | 539233 |
+| test_changeSocializingPoolStateWithSomeCoolDown(string,uint64,uint64) | 539231 |
|------------------------------------------------------------------------|--------|
|                                                                        |     -2 |

The average saved gas is 2.
```
                

```diff
|   Test Name                                   | Gas     |
|-----------------------------------------------|---------|
-| test_getAllActiveValidators(uint256,uint256) | 2157646 |
+| test_getAllActiveValidators(uint256,uint256) | 2154251 |
|-----------------------------------------------|---------|
|                                               |   -3395 |

The average saved gas is 3395.
```
                

```diff
|   Test Name                 | Gas     |
|-----------------------------|---------|
-| test_withdrawnValidators() | 2150340 |
+| test_withdrawnValidators() | 2134868 |
|-----------------------------|---------|
|                             |  -15472 |

The saved gas is 15472.
```
                

```diff
|   Test Name                                                               | Gas   |
|---------------------------------------------------------------------------|-------|
-| testFail_depositSDAsCollateral_withInsufficientApproval(uint256,uint256) | 87525 |
+| testFail_depositSDAsCollateral_withInsufficientApproval(uint256,uint256) | 87518 |
|---------------------------------------------------------------------------|-------|
|                                                                           |    -7 |

The average saved gas is 7.
```
                

```diff
|   Test Name                    | Gas     |
|--------------------------------|---------|
-| test_JustToIncreaseCoverage() | 2994861 |
+| test_JustToIncreaseCoverage() | 2968403 |
|--------------------------------|---------|
|                                |  -26458 |

The saved gas is 26458.
```
                

```diff
|   Test Name                                         | Gas    |
|-----------------------------------------------------|--------|
-| test_depositSDAsCollateral(uint128,uint128,uint16) | 137822 |
+| test_depositSDAsCollateral(uint128,uint128,uint16) | 137782 |
|-----------------------------------------------------|--------|
|                                                     |    -40 |

The average saved gas is 40.
```
                

```diff
|   Test Name                                 | Gas    |
|---------------------------------------------|--------|
-| test_getRemainingSDToBond(uint256,uint256) | 245742 |
+| test_getRemainingSDToBond(uint256,uint256) | 245590 |
|---------------------------------------------|--------|
|                                             |   -152 |

The average saved gas is 152.
```
                

```diff
|   Test Name                        | Gas    |
|------------------------------------|--------|
-| test_getRewardEligibleSD(uint256) | 251745 |
+| test_getRewardEligibleSD(uint256) | 251569 |
|------------------------------------|--------|
|                                    |   -176 |

The average saved gas is 176.
```
                

```diff
|   Test Name                      | Gas   |
|----------------------------------|-------|
-| test_sd_eth_converters(uint128) | 40739 |
+| test_sd_eth_converters(uint128) | 40674 |
|----------------------------------|-------|
|                                  |   -65 |

The average saved gas is 65.
```
                

```diff
|   Test Name              | Gas    |
|--------------------------|--------|
-| test_slashValidatorSD() | 696615 |
+| test_slashValidatorSD() | 696393 |
|--------------------------|--------|
|                          |   -222 |

The saved gas is 222.
```
                

```diff
|   Test Name                                                    | Gas    |
|----------------------------------------------------------------|--------|
-| test_slashValidatorSD_auctionLotNotCreated_whenNoCollateral() | 213843 |
+| test_slashValidatorSD_auctionLotNotCreated_whenNoCollateral() | 213794 |
|----------------------------------------------------------------|--------|
|                                                                |    -49 |

The saved gas is 49.
```
                

```diff
|   Test Name                     | Gas    |
|---------------------------------|--------|
-| test_withdraw(uint128,uint128) | 261483 |
+| test_withdraw(uint128,uint128) | 261322 |
|---------------------------------|--------|
|                                 |   -161 |

The average saved gas is 161.
```
                

```diff
|   Test Name                                         | Gas   |
|-----------------------------------------------------|-------|
-| test_withdraw_revertIfPoolThresholdNotSet(uint256) | 48627 |
+| test_withdraw_revertIfPoolThresholdNotSet(uint256) | 48633 |
|-----------------------------------------------------|-------|
|                                                     |     6 |

The average lost gas is 6.
```
                

```diff
|   Test Name                                                      | Gas    |
|------------------------------------------------------------------|--------|
-| test_withdraw_reverts_InsufficientSDToWithdraw(uint128,uint128) | 247885 |
+| test_withdraw_reverts_InsufficientSDToWithdraw(uint128,uint128) | 247783 |
|------------------------------------------------------------------|--------|
|                                                                  |   -102 |

The average saved gas is 102.
```
                

| Test                           | Name                                                                                 | Gas     |
|--------------------------------|--------------------------------------------------------------------------------------|---------|
| NodeELRewardVaultTest          | test_withdraw(uint128)                                                               | -2739   |
| PenaltyTest                    | test_JustToIncreaseCoverage()                                                        | -6825   |
| PenaltyTest                    | test_additionalPenaltyAmount(address,uint256,bytes)                                  | 6       |
| PenaltyTest                    | test_calculateMEVTheftPenalty()                                                      | -38     |
| PenaltyTest                    | test_updateTotalPenaltyAmount()                                                      | -94     |
| PermissionedNodeRegistryTest   | test_ActivateNodeOperator(uint64,uint64)                                             | -6      |
| PermissionedNodeRegistryTest   | test_AllocateValidatorsAndUpdateOperatorId()                                         | -12     |
| PermissionedNodeRegistryTest   | test_DeactivateNodeOperator(uint64,uint64)                                           | -6      |
| PermissionedNodeRegistryTest   | test_JustToIncreaseCoverage()                                                        | -200    |
| PermissionedNodeRegistryTest   | test_OnboardOperator(string,uint64,uint64)                                           | -6      |
| PermissionedNodeRegistryTest   | test_addValidatorKeys()                                                              | -6      |
| PermissionedNodeRegistryTest   | test_addValidatorKeysOPCrossingMaxNonTerminalKeys()                                  | -6      |
| PermissionedNodeRegistryTest   | test_addValidatorKeysWithInsufficientSDCollateral()                                  | -6      |
| PermissionedNodeRegistryTest   | test_addValidatorKeysWithInvalidKeyCount()                                           | -6      |
| PermissionedNodeRegistryTest   | test_addValidatorKeysWithMisMatchingInputs()                                         | -6      |
| PermissionedNodeRegistryTest   | test_getAllActiveValidators(uint256,uint256)                                         | -10110  |
| PermissionedNodeRegistryTest   | test_getAllActiveValidatorsWithZeroPageNumber(uint256)                               | -6      |
| PermissionedNodeRegistryTest   | test_getOperatorTotalNonTerminalKeys(uint64,uint64,uint256,uint256)                  | -6      |
| PermissionedNodeRegistryTest   | test_getOperatorTotalNonTerminalKeysInvalidPagination(uint64,uint64,uint256,uint256) | -6      |
| PermissionedNodeRegistryTest   | test_getValidatorsByOperator()                                                       | -6      |
| PermissionedNodeRegistryTest   | test_markReadyToDepositValidator()                                                   | -6      |
| PermissionedNodeRegistryTest   | test_updateNextQueuedValidatorIndex(uint64,uint256)                                  | -6      |
| PermissionedNodeRegistryTest   | test_updateOperatorDetailWithInvalidName(string,uint64,uint64,uint64)                | -6      |
| PermissionedNodeRegistryTest   | test_updateOperatorDetailWithZeroRewardAddr(string,uint64)                           | -6      |
| PermissionedNodeRegistryTest   | test_updateOperatorDetails(string,uint64,uint64,uint64)                              | -6      |
| PermissionedNodeRegistryTest   | test_withdrawnValidators()                                                           | -6      |
| PermissionlessNodeRegistryTest | test_JustToIncreaseCoverage()                                                        | -109695 |
| PermissionlessNodeRegistryTest | test_OnboardOperatorWhenPaused(string,uint64,uint64)                                 | -1      |
| PermissionlessNodeRegistryTest | test_changeSocializingPoolStateWithSomeCoolDown(string,uint64,uint64)                | -2      |
| PermissionlessNodeRegistryTest | test_getAllActiveValidators(uint256,uint256)                                         | -3395   |
| PermissionlessNodeRegistryTest | test_withdrawnValidators()                                                           | -15472  |
| SDCollateralTest               | testFail_depositSDAsCollateral_withInsufficientApproval(uint256,uint256)             | -7      |
| SDCollateralTest               | test_JustToIncreaseCoverage()                                                        | -26458  |
| SDCollateralTest               | test_depositSDAsCollateral(uint128,uint128,uint16)                                   | -40     |
| SDCollateralTest               | test_getRemainingSDToBond(uint256,uint256)                                           | -152    |
| SDCollateralTest               | test_getRewardEligibleSD(uint256)                                                    | -176    |
| SDCollateralTest               | test_sd_eth_converters(uint128)                                                      | -65     |
| SDCollateralTest               | test_slashValidatorSD()                                                              | -222    |
| SDCollateralTest               | test_slashValidatorSD_auctionLotNotCreated_whenNoCollateral()                        | -49     |
| SDCollateralTest               | test_withdraw(uint128,uint128)                                                       | -161    |
| SDCollateralTest               | test_withdraw_revertIfPoolThresholdNotSet(uint256)                                   | 6       |
| SDCollateralTest               | test_withdraw_reverts_InsufficientSDToWithdraw(uint128,uint128)                      | -102    |
| --                             | --                                                                                   |         |
| Total                          |                                                                                      | -176111 |

## GO-03
### Avoid useless checks
#### Description
validatorRegistry[0] cannot have status INIZIALIZED


```diff
File: PermissionlessNodeRegistry.sol

  
    function onlyInitializedValidator(uint256 _validatorId) internal view {
-       if (_validatorId == 0 || validatorRegistry[_validatorId].status != ValidatorStatus.INITIALIZED) {
+       if (validatorRegistry[_validatorId].status != ValidatorStatus.INITIALIZED) {
            revert UNEXPECTED_STATUS();
        }
    }

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L712-L715](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L712-L715)


#### Gas Report:
```
Using forge snapshot
```

```diff
|   Test Name                    | Gas     |
|--------------------------------|---------|
-| test_JustToIncreaseCoverage() | 5531119 |
+| test_JustToIncreaseCoverage() | 5528910 |
|--------------------------------|---------|
|                                |   -2209 |

The saved gas is 2209.
```
                

```diff
|   Test Name                                           | Gas   |
|-------------------------------------------------------|-------|
-| test_OnboardOperatorWhenPaused(string,uint64,uint64) | 50497 |
+| test_OnboardOperatorWhenPaused(string,uint64,uint64) | 50496 |
|-------------------------------------------------------|-------|
|                                                       |    -1 |

The average saved gas is 1.
```
                

```diff
|   Test Name                                   | Gas     |
|-----------------------------------------------|---------|
-| test_getAllActiveValidators(uint256,uint256) | 2157646 |
+| test_getAllActiveValidators(uint256,uint256) | 2149643 |
|-----------------------------------------------|---------|
|                                               |   -8003 |

The average saved gas is 8003.
```
                

```diff
|   Test Name                                             | Gas     |
|---------------------------------------------------------|---------|
-| test_getAllActiveValidatorsWithZeroPageNumber(uint256) | 1981021 |
+| test_getAllActiveValidatorsWithZeroPageNumber(uint256) | 1980990 |
|---------------------------------------------------------|---------|
|                                                         |     -31 |

The average saved gas is 31.
```
                

```diff
|   Test Name                                                    | Gas     |
|----------------------------------------------------------------|---------|
-| test_getNodeELVaultAddressForOptOutOperators(uint256,uint256) | 1228311 |
+| test_getNodeELVaultAddressForOptOutOperators(uint256,uint256) | 1228370 |
|----------------------------------------------------------------|---------|
|                                                                |      59 |

The average lost gas is 59.
```
                

```diff
|   Test Name                         | Gas     |
|-------------------------------------|---------|
-| test_markReadyToDepositValidator() | 2082133 |
+| test_markReadyToDepositValidator() | 2082009 |
|-------------------------------------|---------|
|                                     |    -124 |

The saved gas is 124.
```
                

| Test                           | Name                                                          | Gas    |
|--------------------------------|---------------------------------------------------------------|--------|
| PermissionlessNodeRegistryTest | test_JustToIncreaseCoverage()                                 | -2209  |
| PermissionlessNodeRegistryTest | test_OnboardOperatorWhenPaused(string,uint64,uint64)          | -1     |
| PermissionlessNodeRegistryTest | test_getAllActiveValidators(uint256,uint256)                  | -8003  |
| PermissionlessNodeRegistryTest | test_getAllActiveValidatorsWithZeroPageNumber(uint256)        | -31    |
| PermissionlessNodeRegistryTest | test_getNodeELVaultAddressForOptOutOperators(uint256,uint256) | 59     |
| PermissionlessNodeRegistryTest | test_markReadyToDepositValidator()                            | -124   |
| --                             | --                                                            |        |
| Total                          |                                                               | -10309 |