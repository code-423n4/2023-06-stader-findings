|     |issue |Instance
|---- |----- |--------
|[G-1]|bytes constants are more eficient than string constans|9
|[G-2]|Use nested if and, avoid multiple check combinations|5
|[G-3]|Use calldata instead of memory for function arguments that do not get mutated|13
|[G-4]|Use assembly for loops|10
|[G-5]|A modifier used only once and not being inherted should be inlined to save gas|3
|[G-6]|Use ERC721A instead ERC721|7
|[G-7]|++i costs less gas than i++, especially when it’s used n for-loops (--i/i-- too)|14
|[G-8]|bytes constants are more eficient than string constans|6
|[G-9]|Emit can be rearranged|6
|[G-10]|Use hardcode address instead address(this)|18
|[G-11]|Unnecessary look up in if condition|3
|[G-12]|Avoid emitting constants.|4
1--------------------------------------------------------------------------
### [G-1] bytes constants are more eficient than string constans
If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.
,,,solidity
106     function onboardNodeOperator(string calldata _operatorName, address payable _operatorRewardAddress)
401        function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external       override   
661   function onboardOperator(string calldata _operatorName, address payable _operatorRewardAddress) internal   
91     string calldata _operatorName,
366     function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override 
600     string calldata _operatorName,
215      function onlyValidName(string calldata _name) external view 
217   revert EmptyNameString();
124   string memory _units
,,,
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#LL106C34-L106C40
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L401
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L661
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L91
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L366
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L600
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L215
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L217
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L124

2---------------------------------------------------------------------------------
### [G-2] Use nested if and, avoid multiple check combinations
Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.
,,,solidity
514  if (
            !(variablesMap[MIN_DEPOSIT_AMOUNT] != 0 &&
515 variablesMap[MIN_WITHDRAW_AMOUNT] != 0 &&
516 variablesMap[MIN_DEPOSIT_AMOUNT] <= variablesMap[MAX_DEPOSIT_AMOUNT] &&
517  variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MAX_WITHDRAW_AMOUNT] &&
518    variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MIN_DEPOSIT_AMOUNT] &&

,,,
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L514
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L515
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L516
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L517
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L518

3--------------------------------------------------------------------------------------
### [G-3] Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.
,,,solidity
195   returns (uint256[] memory selectedOperatorCapacity)
202     uint256[] memory remainingOperatorCapacity = new uint256[](nextOperatorId);
588   returns (Validator[] memory)
596  Validator[] memory validators = new Validator[](_pageSize);
95  uint256[] memory selectedOperatorCapacity = IPermissionedNodeRegistry(nodeRegistryAddress)
79     returns (uint256[] memory selectedPoolCapacity, uint8[] memory poolIdArray)
121  uint8[] memory poolIdArray = IPoolUtils(staderConfig.getPoolUtils()).getPoolIdArray();
210 function getPoolIdArray() external view override returns (uint8[] memory) 
124      string memory _units
227     (uint256[] memory selectedPoolCapacity, uint8[] memory poolIdArray) = IPoolSelector
133 UserWithdrawInfo memory userWithdrawInfo = userWithdrawRequests[requestId];
184  function getRequestIdsByUser(address _owner) external view override returns (uint256[] memory) 
54  internal view returns (bytes memory) 
,,,
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L195
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L202
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L588
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L596
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L95
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L79
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L121
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L210
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L124
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L227
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L133
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L184
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L54
4---------------------------------------------------------------------------------------------------
### [G-4] Use assembly for loops
In the following instances, assembly is used for more gas efficient loops. 
,,,solidity
100      for (uint256 i; i < reportedValidatorCount; ) {
            if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
                revert ValidatorSettled();
            }
91         for (uint256 i = 0; i < permissionedNosLength; i++) {
            address operator = _permissionedNOs[i];
            UtilLib.checkNonZeroAddress(operator);
            permissionList[operator] = true;
            emit OperatorWhitelisted(operator);
        }
    }
206             for (uint256 i = 1; i < nextOperatorId; ) {
                if (!operatorStructById[i].active) {
                    continue;
                }
                remainingOperatorCapacity[i] = getOperatorQueuedValidatorCount(i);
                selectedOperatorCapacity[i] = Math.min(remainingOperatorCapacity[i], validatorPerOperator);
                totalValidatorToDeposit += selectedOperatorCapacity[i];
                remainingOperatorCapacity[i] -= selectedOperatorCapacity[i];
                unchecked {
                    ++i;
                }
            }
 272        for (uint256 i; i < frontRunValidatorsLength; ) {
            uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];
            // only PRE_DEPOSIT status check will also include validatorId = 0 check
            // as status for that will be INITIALIZED(default status)
            onlyPreDepositValidator(validatorId);
            handleFrontRun(validatorId);
            emit ValidatorMarkedAsFrontRunned(_frontRunPubkey[i], validatorId);
            unchecked {
                ++i;
            }
        }
285         for (uint256 i; i < invalidSignatureValidatorsLength; ) {
            uint256 validatorId = validatorIdByPubkey[_invalidSignaturePubkey[i]];
            // only PRE_DEPOSIT status check will also include validatorId = 0 check
            // as status for that will be INITIALIZED(default status)
            onlyPreDepositValidator(validatorId);
            validatorRegistry[validatorId].status = ValidatorStatus.INVALID_SIGNATURE;
            emit ValidatorStatusMarkedAsInvalidSignature(_invalidSignaturePubkey[i], validatorId);
            unchecked {
                ++i;
            }
        }
317        for (uint256 i; i < withdrawnValidatorCount; ) {
            uint256 validatorId = validatorIdByPubkey[_pubkeys[i]];
            if (validatorRegistry[validatorId].status != ValidatorStatus.DEPOSITED) {
                revert UNEXPECTED_STATUS();
            }
            validatorRegistry[validatorId].status = ValidatorStatus.WITHDRAWN;
            validatorRegistry[validatorId].withdrawnBlock = block.number;
            IValidatorWithdrawalVault(validatorRegistry[validatorId].withdrawVaultAddress).settleFunds();
            emit ValidatorWithdrawn(_pubkeys[i], validatorId);
            unchecked {
                ++i;
            }
        }
488         for (uint256 i = 1; i < nextOperatorId; ) {
            if (operatorStructById[i].active) {
                totalQueuedValidators += getOperatorQueuedValidatorCount(i);
            }
            unchecked {
                ++i;
            }
        }
        return totalQueuedValidators;
534       for (uint256 i = _startIndex; i < _endIndex; ) {
            uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
            if (isNonTerminalValidator(validatorId)) {
                totalNonWithdrawnKeyCount++;
            }
            unchecked {
                ++i;
            }
        }
        return totalNonWithdrawnKeyCount;
100       for (uint256 i = 1; i < selectedOperatorCapacityLength; ) {
            uint256 validatorToDeposit = selectedOperatorCapacity[i];
            if (validatorToDeposit == 0) {
                continue;
            }
109           for (
                uint256 index = nextQueuedValidatorIndex;
                index < nextQueuedValidatorIndex + validatorToDeposit;
                index++
            ) {
                uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdsByOperatorId(i, index);
                preDepositOnBeaconChain(nodeRegistryAddress, vaultFactory, ethDepositContract, validatorId);
            }
            IPermissionedNodeRegistry(nodeRegistryAddress).updateQueuedValidatorIndex(
                i,
                nextQueuedValidatorIndex + validatorToDeposit
            );
            unchecked {
                ++i;
            }

,,,
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L100
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L91
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L206
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L272
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L285
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L317
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L488
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L534
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L100
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L109

5----------------------------------------------------------------------------------------------------

### [G‑5] A modifier used only once and not being inherited should be inlined to save gas
,,,solidity
  697  modifier checkERInspectionMode() {
        if (erInspectionMode) {
            revert InspectionModeActive();
        }
        _;
    }

   704 modifier trustedNodeOnly() {
        if (!isTrustedNode[msg.sender]) {
            revert NotATrustedNode();
        }
        _;
    }

   711 modifier checkMinTrustedNodes() {
        if (trustedNodesCount < MIN_TRUSTED_NODES) {
            revert InsufficientTrustedNodes();
        }
        _;
,,,}
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L697--l-715

6--------------------------------------------------------------------------------------------------
### [G-6] Use ERC721A instead ERC721
ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee.
,,,solidity
 14  import '@openzeppelin/contracts/token/ERC20/IERC20.sol';
 15 import '@openzeppelin/contracts/token/ERC20/IERC20.sol';
 13 import '@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol';
 26   using SafeERC20Upgradeable for IERC20Upgradeable;
 9 import '@openzeppelin/contracts/token/ERC20/IERC20.sol';
 7 import '@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol';
 17 contract ETHx is Initializable, ERC20Upgradeable, PausableUpgradeable, AccessControlUpgradeable
 ,,, 
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L14
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L15
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L13
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L26
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L9
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L7
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L17
7----------------------------------------------------------------------------------------------
### [G‑7] ++i costs less gas than i++, especially when it’s used in for-loops (--i/i-- too)

Saves 5 gas per loop
,,,solidity
91  for (uint256 i = 0; i < permissionedNosLength; i++) 
598  for (uint256 i = startIndex; i < endIndex; i++)
637   for (uint256 i = startIndex; i < endIndex; i++) 
504  for (uint256 i = startIndex; i < endIndex; i++) 
543   for (uint256 i = startIndex; i < endIndex; i++)
573 for (uint256 i = startIndex; i < endIndex; i++)
130 for (uint256 i = 0; i < poolTargetLength; i++) 
105  for (uint256 i = 0; i < poolCount; i++) 
168    for (uint256 i = 0; i < poolCount; i++)
179   for (uint256 i = 0; i < poolCount; i++)
190 for (uint256 i = 0; i < poolCount; i++) 
273   for (uint256 i = 0; i < poolCount; i++) 
145  for (uint256 i = 0; i < indexLength; i++)
232  for (uint256 i = 0; i < poolCount; i++) 

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L91
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L598
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L637
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L504
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L543
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L573
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L130
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L105
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L168
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L179
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L190
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L273
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L145
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L232
8--------------------------------------------------------------------------------------------------------
### [G-8] bytes constants are more eficient than string constans
If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.
,,,solidity
124  string memory _units
106    function onboardNodeOperator(string calldata _operatorName, address payable _operatorRewardAddress)
401   function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override 
661  function onboardOperator(string calldata _operatorName, address payable _operatorRewardAddress) internal
91  string calldata _operatorName,
        address payable _operatorRewardAddress
600   string calldata _operatorName,
        address payable _operatorRewardAddress

,,,
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L124
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L106
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L401
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L661
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L91
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L600
9---------------------------------------------------------------------------------------------------------
### [G-09] Emit can be rearranged
The emit can be rearranged in the changeNoteAddress() and changeRevenueAddress() functions. With this arrangement, both the deploy time (~1.4 k * 4 = 5.6k gas per function) and every time the functions are triggered (~16 gas) gas savings will be achieved.
,,,solidity
59  emit UpdatedStaderConfig(_staderConfig);
150   emit UpdatedStaderConfig(_staderConfig);
87 emit UpdatedStaderConfig(_staderConfig);
182    emit UpdatedStaderConfig(_staderConfig);
72   emit UpdatedStaderConfig(_staderConfig);
154  emit BidIncrementUpdated(_bidIncrement);
,,,
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L59
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L150
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L87
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L182
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L72
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L154
10-----------------------------------------------------------------------------------------------------------------
### [G-10] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. 
,,,solidity
174     if (preDepositValidatorCount != 0 || address(this).balance == 0) 


178   IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExcessEthFromPool
            value: address(this).balance

47      if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }   

69      address(this).balance - totalOperatorETHRewardsRemaining 

75    IERC20(staderConfig.getStaderToken()).balanceOf(address(this)) - totalOperatorSDRewardsRemaining

43    if (address(this).balance < _amount || _amount == 0) {
            revert InvalidAmountProvided();
        }

 62  if (address(this).balance < _amount) {
            revert InSufficientBalance();       
          }

189 (, uint256 availableETHForNewDeposit) = SafeMath.trySub(
            address(this).balance,    

221  (, uint256 availableETHForNewDeposit) = SafeMath.trySub(
            address(this).balance,   

104  IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);

155  ETHx(staderConfig.getETHxToken()).burnFrom(address(this), lockedEthXToBurn);

224      if (address(this).balance < _amount) {
            revert InSufficientBalance();
        }

31      uint8 poolId = VaultProxy(payable(address(this))).poolId();
        uint256 validatorId = VaultProxy(payable(address(this))).id();
         IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
34       uint256 totalRewards = address(this).balance;

55   uint8 poolId = VaultProxy(payable(address(this))).poolId();
     uint256 validatorId = VaultProxy(payable(address(this))).id();
57    IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();

94  uint8 poolId = VaultProxy(payable(address(this))).poolId();
95  IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();

99      uint256 contractBalance = address(this).balance;

55
        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }

25   uint8 poolId = IVaultProxy(address(this)).poolId();
       uint256 operatorId = IVaultProxy(address(this)).id();
      IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
28        uint256 totalRewards = address(this).balance;        
,,,
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L174
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L178
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L47---L49
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L69
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L75
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L43
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L62
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L189
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L221
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L104
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L155
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L224
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L31---L34
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L55---L57
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L94---L95
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L99
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L55
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L25---28
11-----------------------------------------------------------------------------------------------------------
### [G-11] Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

,,,solidity
697  if (keyCount == 0 || keyCount > inputKeyCountLimit) {
            revert InvalidKeyCount();
             } 

653      if (keyCount == 0 || keyCount > inputKeyCountLimit) {
            revert InvalidKeyCount();
        }

713    if (_validatorId == 0 || validatorRegistry[_validatorId].status != ValidatorStatus.INITIALIZED) {
            revert UNEXPECTED_STATUS();
        }        

,,,
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L697
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L653
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L713
12-----------------------------------------------------------------------------------------------------------
### [G-12] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas)

,,,solidity
139        emit ExchangeRateSubmitted(
            msg.sender,
            _exchangeRate.reportingBlockNumber,
            _exchangeRate.totalETHBalance,
            _exchangeRate.totalETHXSupply,
            block.timestamp
145        );
245      emit SocializingRewardsMerkleRootSubmitted(
            msg.sender,
            _rewardsData.index,
            _rewardsData.merkleRoot,
            _rewardsData.poolId,
            block.number
251        );
261       emit SocializingRewardsMerkleRootUpdated(
                _rewardsData.index,
                _rewardsData.merkleRoot,
                _rewardsData.poolId,
                block.number
266            );

,,,
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L139---L45
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L245---L251
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L261---L266