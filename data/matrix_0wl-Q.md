## Non Critical Issues

|       | Issue                                                                         |
| ----- | :---------------------------------------------------------------------------- |
| NC-1  | ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS                                 |
| NC-2  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                          |
| NC-3  | GENERATE PERFECT CODE HEADERS EVERY TIME                                      |
| NC-4  | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS                               |
| NC-5  | IMPLEMENTATION CONTRACT MAY NOT BE INITIALIZED                                |
| NC-6  | USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’                         |
| NC-7  | MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL                        |
| NC-8  | LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION                         |
| NC-9  | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION                 |
| NC-10 | NO SAME VALUE INPUT CONTROL                                                   |
| NC-11 | Add parameter to event-emit                                                   |
| NC-12 | SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC                            |
| NC-13 | USE A MORE RECENT VERSION OF SOLIDITY                                         |
| NC-14 | Consider renaming this function getVaultFactory() to getVaultFactoryAddress() |
| NC-15 | Functions not used internally could be marked external                        |

### [NC-1] ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS

#### Description:

Add to indexed parameter for countable Events

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

43:         emit UpdatedStaderConfig(_staderConfig);

44:         emit AuctionDurationUpdated(duration);

45:         emit BidIncrementUpdated(bidIncrement);

58:         emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);

77:         emit BidPlaced(lotId, msg.sender, totalUserBid);

90:         emit SDClaimed(lotId, lotItem.highestBidder, lotItem.sdAmount);

103:         emit ETHClaimed(lotId, staderConfig.getStakePoolManager(), ethAmount);

117:         emit UnsuccessfulSDAuctionExtracted(lotId, _sdAmount, staderConfig.getStaderTreasury());

134:         emit BidWithdrawn(lotId, msg.sender, withdrawalAmount);

141:         emit UpdatedStaderConfig(_staderConfig);

148:         emit AuctionDurationUpdated(duration);

154:         emit BidIncrementUpdated(_bidIncrement);

```

```solidity
File: contracts/NodeELRewardVault.sol

21:         emit ETHReceived(msg.sender, msg.value);

44:         emit Withdrawal(protocolShare, operatorShare, userShare);

```

```solidity
File: contracts/OperatorRewardsCollector.sol

37:         emit UpdatedStaderConfig(_staderConfig);

43:         emit DepositedFor(msg.sender, _receiver, msg.value);

53:         emit Claimed(operatorRewardsAddr, amount);

59:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/Penalty.sol

49:         emit UpdatedPenaltyOracleAddress(_ratedOracleAddress);

58:         emit UpdatedAdditionalPenaltyAmount(_pubkey, _amount);

65:         emit UpdatedMEVTheftPenaltyPerStrike(_mevTheftPenaltyPerStrike);

72:         emit UpdatedMissedAttestationPenaltyPerStrike(_missedAttestationPenaltyPerStrike);

79:         emit UpdatedValidatorExitPenaltyThreshold(_validatorExitPenaltyThreshold);

87:         emit UpdatedPenaltyOracleAddress(_ratedOracleAddress);

94:         emit UpdatedStaderConfig(_staderConfig);

114:                 emit ForceExitValidator(_pubkey[i]);

147:         emit ValidatorMarkedAsSettled(pubkey);

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

95:             emit OperatorWhitelisted(operator);

175:             emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId);

278:             emit ValidatorMarkedAsFrontRunned(_frontRunPubkey[i], validatorId);

291:             emit ValidatorStatusMarkedAsInvalidSignature(_invalidSignaturePubkey[i], validatorId);

325:             emit ValidatorWithdrawn(_pubkeys[i], validatorId);

368:         emit UpdatedQueuedValidatorIndex(_operatorId, _nextQueuedValidatorIndex);

380:         emit UpdatedValidatorDepositBlock(_validatorId, block.number);

392:         emit MarkedValidatorStatusAsPreDeposit(_pubkey);

408:         emit UpdatedOperatorDetails(msg.sender, _operatorName, _rewardAddress);

419:         emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);

430:         emit UpdatedInputKeyCountLimit(inputKeyCountLimit);

441:         emit UpdatedVerifiedKeyBatchSize(_verifiedKeysBatchSize);

452:         emit MaxOperatorIdLimitChanged(_maxOperatorId);

462:         emit UpdatedStaderConfig(_staderConfig);

478:         emit IncreasedTotalActiveValidatorCount(totalActiveValidatorCount);

667:         emit OnboardedOperator(msg.sender, _operatorRewardAddress, nextOperatorId - 1);

749:         emit DecreasedTotalActiveValidatorCount(totalActiveValidatorCount);

765:         emit OperatorDeactivated(_operatorId);

771:         emit OperatorActivated(_operatorId);

```

```solidity
File: contracts/PermissionedPool.sol

63:         emit ReceivedInsuranceFund(msg.value);

82:         emit TransferredETHToSSPMForDefectiveKeys(amountToSendToPoolManager);

162:             emit ValidatorDepositedOnBeaconChain(validatorId, _pubkey[i]);

244:         emit UpdatedCommissionFees(_protocolFee, _operatorFee);

251:         emit UpdatedStaderConfig(_staderConfig);

317:         emit ValidatorPreDepositedOnBeaconChain(pubkey);

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

161:             emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId);

203:             emit ValidatorMarkedReadyToDeposit(_readyToDepositPubkey[i], validatorId);

219:             emit ValidatorMarkedAsFrontRunned(_frontRunPubkey[i], validatorId);

229:             emit ValidatorStatusMarkedAsInvalidSignature(_invalidSignaturePubkey[i], validatorId);

255:             emit ValidatorWithdrawn(_pubkeys[i], validatorId);

271:         emit UpdatedNextQueuedValidatorIndex(nextQueuedValidatorIndex);

283:         emit UpdatedValidatorDepositBlock(_validatorId, block.number);

312:         emit UpdatedSocializingPoolState(operatorId, _optInForSocializingPool, block.number);

328:         emit UpdatedInputKeyCountLimit(inputKeyCountLimit);

339:         emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);

350:         emit UpdatedVerifiedKeyBatchSize(_verifiedKeysBatchSize);

357:         emit UpdatedStaderConfig(_staderConfig);

373:         emit UpdatedOperatorDetails(msg.sender, _operatorName, _rewardAddress);

384:         emit IncreasedTotalActiveValidatorCount(totalActiveValidatorCount);

395:         emit TransferredCollateralToPool(_amount);

614:         emit OnboardedOperator(msg.sender, _operatorRewardAddress, nextOperatorId - 1, _optInForSocializingPool);

709:         emit DecreasedTotalActiveValidatorCount(totalActiveValidatorCount);

```

```solidity
File: contracts/PermissionlessPool.sol

62:         emit ReceivedCollateralETH(msg.value);

74:         emit UpdatedCommissionFees(_protocolFee, _operatorFee);

115:             emit ValidatorPreDepositedOnBeaconChain(_pubkey[i]);

213:         emit UpdatedStaderConfig(_staderConfig);

269:         emit ValidatorDepositedOnBeaconChain(_validatorId, pubkey);

```

```solidity
File: contracts/PoolSelector.sol

133:             emit UpdatedPoolWeight(poolIdArray[i], _poolTargets[i]);

143:         emit UpdatedPoolAllocationMaxSize(_poolAllocationMaxSize);

150:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/PoolUtils.sol

46:         emit PoolAdded(_poolId, _poolAddress);

64:         emit PoolAddressUpdated(_poolId, _newPoolAddress);

76:             emit ExitValidator(_pubkeys[i]);

87:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/SDCollateral.sol

36:         emit UpdatedStaderConfig(_staderConfig);

51:         emit SDDeposited(operator, _sdAmount);

72:         emit SDWithdrawn(operator, _requestedSD);

98:         emit SDSlashed(_operator, staderConfig.getAuctionContract(), sdSlashed);

116:         emit UpdatedStaderConfig(_staderConfig);

138:         emit UpdatedPoolThreshold(_poolId, _minThreshold, _withdrawThreshold);

```

```solidity
File: contracts/SocializingPool.sol

58:         emit ETHReceived(msg.sender, msg.value);

96:         emit OperatorRewardsUpdated(

103:         emit UserETHRewardsTransferred(_rewardsData.userETHRewards);

104:         emit ProtocolETHRewardsTransferred(_rewardsData.protocolETHRewards);

134:         emit OperatorRewardsClaimed(operatorRewardsAddr, totalAmountETH, totalAmountSD);

182:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/StaderConfig.sol

473:         emit SetConstant(key, val);

478:         emit SetConstant(key, val);

484:         emit SetAccount(key, val);

490:         emit SetContract(key, val);

496:         emit SetToken(key, val);

```

```solidity
File: contracts/StaderInsuranceFund.sol

37:         emit ReceivedInsuranceFund(msg.value);

52:         emit FundWithdrawn(_amount);

72:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/StaderOracle.sol

77:         emit UpdatedStaderConfig(_staderConfig);

90:         emit TrustedNodeAdded(_nodeAddress);

103:         emit TrustedNodeRemoved(_nodeAddress);

139:         emit ExchangeRateSubmitted(

245:         emit SocializingRewardsMerkleRootSubmitted(

261:             emit SocializingRewardsMerkleRootUpdated(

287:         emit SDPriceSubmitted(msg.sender, _sdPriceData.sdPriceInETH, _sdPriceData.reportingBlockNumber, block.number);

296:             emit SDPriceUpdated(_sdPriceData.sdPriceInETH, _sdPriceData.reportingBlockNumber, block.number);

359:         emit ValidatorStatsSubmitted(

378:             emit ValidatorStatsUpdated(

423:         emit WithdrawnValidatorsSubmitted(

439:             emit WithdrawnValidatorsUpdated(

474:         emit MissedAttestationPenaltySubmitted(

492:             emit MissedAttestationPenaltyUpdated(_mapd.index, block.number, _mapd.sortedPubkeys);

500:         emit SafeModeEnabled();

505:         emit SafeModeDisabled();

512:         emit UpdatedStaderConfig(_staderConfig);

526:         emit ERDataSourceToggled(isPORFeedBasedERData);

536:         emit UpdatedERChangeLimit(erChangeLimit);

568:         emit UpdateFrequencyUpdated(_updateFrequency);

673:             emit ERInspectionModeActivated(erInspectionMode, block.timestamp);

689:         emit ExchangeRateUpdated(

```

```solidity
File: contracts/StaderStakePoolsManager.sol

74:         emit ExecutionLayerRewardsReceived(msg.value);

79:         emit WithdrawVaultUserShareReceived(msg.value);

83:         emit AuctionedEthReceived(msg.value);

91:         emit ReceivedExcessEthFromPool(_poolId);

106:         emit TransferredETHToUserWithdrawManager(_amount);

112:         emit UpdatedExcessETHDepositCoolDown(_excessETHDepositCoolDown);

119:         emit UpdatedStaderConfig(_staderConfig);

208:         emit ETHTransferredToPool(_poolId, poolAddress, selectedPoolCapacity * poolDepositSize);

244:             emit ETHTransferredToPool(i, poolAddress, validatorToDeposit * poolDepositSize);

322:         emit Deposited(_caller, _receiver, _assets, _shares);

```

```solidity
File: contracts/UserWithdrawalManager.sol

69:         emit ReceivedETH(msg.value);

80:         emit UpdatedFinalizationBatchLimit(_finalizationBatchLimit);

87:         emit UpdatedStaderConfig(_staderConfig);

108:         emit WithdrawRequestReceived(msg.sender, _owner, nextRequestId, _ethXAmount, assets);

157:             emit FinalizedWithdrawRequest(requestId);

180:         emit RequestRedeemed(msg.sender, userRequest.owner, etherToTransfer);

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

27:         emit ETHReceived(msg.sender, msg.value);

36:             emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());

51:         emit DistributedRewards(userShare, operatorShare, protocolShare);

82:         emit SettledFunds(userShare, operatorShare, protocolShare);

```

```solidity
File: contracts/VaultProxy.sol

60:         emit UpdatedStaderConfig(_staderConfig);

71:         emit UpdatedOwner(owner);

```

```solidity
File: contracts/factory/VaultFactory.sol

44:         emit WithdrawVaultCreated(withdrawVaultAddress);

58:         emit NodeELRewardVaultCreated(nodeELRewardVaultAddress);

89:         emit UpdatedStaderConfig(_staderConfig);

96:         emit UpdatedVaultProxyImplementation(vaultProxyImplementation);

```

### [NC-2] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: contracts/Penalty.sol

53:     function setAdditionalPenaltyAmount(bytes calldata _pubkey, uint256 _amount) external override {

```

```solidity
File: contracts/PermissionedPool.sol

236:     function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {

```

```solidity
File: contracts/PermissionlessPool.sol

66:     function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {

```

```solidity
File: contracts/StaderConfig.sol

89:         setConstant(ETH_PER_NODE, 32 ether);

90:         setConstant(PRE_DEPOSIT_SIZE, 1 ether);

91:         setConstant(FULL_DEPOSIT_SIZE, 31 ether);

92:         setConstant(TOTAL_FEE, 10000);

93:         setConstant(DECIMALS, 10**18);

94:         setConstant(OPERATOR_MAX_NAME_LENGTH, 255);

95:         setVariable(MIN_DEPOSIT_AMOUNT, 10**14);

96:         setVariable(MAX_DEPOSIT_AMOUNT, 10000 ether);

97:         setVariable(MIN_WITHDRAW_AMOUNT, 10**14);

98:         setVariable(MAX_WITHDRAW_AMOUNT, 10000 ether);

99:         setVariable(WITHDRAWN_KEYS_BATCH_SIZE, 50);

100:         setVariable(MIN_BLOCK_DELAY_TO_FINALIZE_WITHDRAW_REQUEST, 600);

101:         setContract(ETH_DEPOSIT_CONTRACT, _ethDepositContract);

108:         setVariable(SOCIALIZING_POOL_CYCLE_DURATION, _socializingPoolCycleDuration);

115:         setVariable(SOCIALIZING_POOL_OPT_IN_COOLING_PERIOD, _SocializePoolOptInCoolingPeriod);

119:         setVariable(REWARD_THRESHOLD, _rewardsThreshold);

127:         setVariable(MIN_DEPOSIT_AMOUNT, _minDepositAmount);

136:         setVariable(MAX_DEPOSIT_AMOUNT, _maxDepositAmount);

145:         setVariable(MIN_WITHDRAW_AMOUNT, _minWithdrawAmount);

154:         setVariable(MAX_WITHDRAW_AMOUNT, _maxWithdrawAmount);

162:         setVariable(MIN_BLOCK_DELAY_TO_FINALIZE_WITHDRAW_REQUEST, _minBlockDelay);

171:         setVariable(WITHDRAWN_KEYS_BATCH_SIZE, _withdrawnKeysBatchSize);

180:         setAccount(ADMIN, _admin);

186:         setAccount(STADER_TREASURY, _staderTreasury);

192:         setContract(POOL_UTILS, _poolUtils);

196:         setContract(POOL_SELECTOR, _poolSelector);

200:         setContract(SD_COLLATERAL, _sdCollateral);

204:         setContract(OPERATOR_REWARD_COLLECTOR, _operatorRewardsCollector);

208:         setContract(VAULT_FACTORY, _vaultFactory);

212:         setContract(AUCTION_CONTRACT, _auctionContract);

216:         setContract(STADER_ORACLE, _staderOracle);

220:         setContract(PENALTY_CONTRACT, _penaltyContract);

224:         setContract(PERMISSIONED_POOL, _permissionedPool);

228:         setContract(STAKE_POOL_MANAGER, _stakePoolManager);

232:         setContract(PERMISSIONLESS_POOL, _permissionlessPool);

236:         setContract(USER_WITHDRAW_MANAGER, _userWithdrawManager);

240:         setContract(STADER_INSURANCE_FUND, _staderInsuranceFund);

244:         setContract(PERMISSIONED_NODE_REGISTRY, _permissionedNodeRegistry);

251:         setContract(PERMISSIONLESS_NODE_REGISTRY, _permissionlessNodeRegistry);

258:         setContract(PERMISSIONED_SOCIALIZING_POOL, _permissionedSocializePool);

265:         setContract(PERMISSIONLESS_SOCIALIZING_POOL, _permissionlessSocializePool);

269:         setContract(NODE_EL_REWARD_VAULT_IMPLEMENTATION, _nodeELRewardVaultImpl);

276:         setContract(VALIDATOR_WITHDRAWAL_VAULT_IMPLEMENTATION, _validatorWithdrawalVaultImpl);

280:         setContract(ETH_BALANCE_POR_FEED, _ethBalanceProxy);

284:         setContract(ETHX_SUPPLY_POR_FEED, _ethXSupplyProxy);

288:         setToken(SD, _staderToken);

292:         setToken(ETHx, _ethX);

471:     function setConstant(bytes32 key, uint256 val) internal {

476:     function setVariable(bytes32 key, uint256 val) internal {

481:     function setAccount(bytes32 key, address val) internal {

487:     function setContract(bytes32 key, address val) internal {

493:     function setToken(bytes32 key, address val) internal {

```

```solidity
File: contracts/StaderOracle.sol

70:         setUpdateFrequency(ETHX_ER_UF, 7200);

71:         setUpdateFrequency(SD_PRICE_UF, 7200);

72:         setUpdateFrequency(VALIDATOR_STATS_UF, 7200);

73:         setUpdateFrequency(WITHDRAWN_VALIDATORS_UF, 14400);

74:         setUpdateFrequency(MISSED_ATTESTATION_PENALTY_UF, 50400);

515:     function setERUpdateFrequency(uint256 _updateFrequency) external override {

520:         setUpdateFrequency(ETHX_ER_UF, _updateFrequency);

539:     function setSDPriceUpdateFrequency(uint256 _updateFrequency) external override {

541:         setUpdateFrequency(SD_PRICE_UF, _updateFrequency);

544:     function setValidatorStatsUpdateFrequency(uint256 _updateFrequency) external override {

546:         setUpdateFrequency(VALIDATOR_STATS_UF, _updateFrequency);

549:     function setWithdrawnValidatorsUpdateFrequency(uint256 _updateFrequency) external override {

551:         setUpdateFrequency(WITHDRAWN_VALIDATORS_UF, _updateFrequency);

554:     function setMissedAttestationPenaltyUpdateFrequency(uint256 _updateFrequency) external override {

556:         setUpdateFrequency(MISSED_ATTESTATION_PENALTY_UF, _updateFrequency);

559:     function setUpdateFrequency(bytes32 _key, uint256 _updateFrequency) internal {

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

54:     function settleFunds() external override {

```

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-4] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS

#### Description:

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

22:     uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours

```

```solidity
File: contracts/ETHx.sol

21:     bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');

22:     bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

31:     uint8 public constant override POOL_ID = 2;

```

```solidity
File: contracts/PermissionedPool.sol

33:     uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

30:     uint8 public constant override POOL_ID = 1;

42:     uint256 public constant override FRONT_RUN_PENALTY = 3 ether;

43:     uint256 public constant COLLATERAL_ETH = 4 ether;

```

```solidity
File: contracts/PermissionlessPool.sol

23:     uint256 public constant DEPOSIT_NODE_BOND = 3 ether;

31:     uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

```

```solidity
File: contracts/PoolSelector.sol

21:     uint256 public constant POOL_WEIGHTS_SUM = 10000;

```

```solidity
File: contracts/StaderConfig.sol

12:     bytes32 public constant ETH_PER_NODE = keccak256('ETH_PER_NODE');

14:     bytes32 public constant PRE_DEPOSIT_SIZE = keccak256('PRE_DEPOSIT_SIZE');

16:     bytes32 public constant FULL_DEPOSIT_SIZE = keccak256('FULL_DEPOSIT_SIZE');

18:     bytes32 public constant DECIMALS = keccak256('DECIMALS');

20:     bytes32 public constant TOTAL_FEE = keccak256('TOTAL_FEE');

22:     bytes32 public constant OPERATOR_MAX_NAME_LENGTH = keccak256('OPERATOR_MAX_NAME_LENGTH');

24:     bytes32 public constant SOCIALIZING_POOL_CYCLE_DURATION = keccak256('SOCIALIZING_POOL_CYCLE_DURATION');

25:     bytes32 public constant SOCIALIZING_POOL_OPT_IN_COOLING_PERIOD =

27:     bytes32 public constant REWARD_THRESHOLD = keccak256('REWARD_THRESHOLD');

28:     bytes32 public constant MIN_DEPOSIT_AMOUNT = keccak256('MIN_DEPOSIT_AMOUNT');

29:     bytes32 public constant MAX_DEPOSIT_AMOUNT = keccak256('MAX_DEPOSIT_AMOUNT');

30:     bytes32 public constant MIN_WITHDRAW_AMOUNT = keccak256('MIN_WITHDRAW_AMOUNT');

31:     bytes32 public constant MAX_WITHDRAW_AMOUNT = keccak256('MAX_WITHDRAW_AMOUNT');

33:     bytes32 public constant MIN_BLOCK_DELAY_TO_FINALIZE_WITHDRAW_REQUEST =

35:     bytes32 public constant WITHDRAWN_KEYS_BATCH_SIZE = keccak256('WITHDRAWN_KEYS_BATCH_SIZE');

37:     bytes32 public constant ADMIN = keccak256('ADMIN');

38:     bytes32 public constant STADER_TREASURY = keccak256('STADER_TREASURY');

40:     bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');

41:     bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');

42:     bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');

43:     bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');

44:     bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');

45:     bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');

46:     bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');

47:     bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');

48:     bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');

49:     bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');

50:     bytes32 public constant override ETH_DEPOSIT_CONTRACT = keccak256('ETH_DEPOSIT_CONTRACT');

51:     bytes32 public constant override PERMISSIONLESS_POOL = keccak256('PERMISSIONLESS_POOL');

52:     bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');

53:     bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');

54:     bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');

55:     bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');

56:     bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');

57:     bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');

58:     bytes32 public constant override NODE_EL_REWARD_VAULT_IMPLEMENTATION =

60:     bytes32 public constant override VALIDATOR_WITHDRAWAL_VAULT_IMPLEMENTATION =

64:     bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');

65:     bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');

68:     bytes32 public constant override MANAGER = keccak256('MANAGER');

69:     bytes32 public constant override OPERATOR = keccak256('OPERATOR');

71:     bytes32 public constant SD = keccak256('SD');

72:     bytes32 public constant ETHx = keccak256('ETHx');

```

```solidity
File: contracts/StaderOracle.sol

26:     uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days

27:     uint256 public constant ER_CHANGE_MAX_BPS = 10000;

29:     uint256 public constant MIN_TRUSTED_NODES = 5;

50:     bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency

51:     bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key

52:     bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key

53:     bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key

54:     bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key

```

```solidity
File: contracts/factory/VaultFactory.sol

16:     bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');

```

### [NC-5] IMPLEMENTATION CONTRACT MAY NOT BE INITIALIZED

#### Description:

OpenZeppelin recommends that the initializer modifier be applied to constructors.

Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.

https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

14: contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableUpgradeable, ReentrancyGuardUpgradeable {

```

```solidity
File: contracts/ETHx.sol

17: contract ETHx is Initializable, ERC20Upgradeable, PausableUpgradeable, AccessControlUpgradeable {

```

```solidity
File: contracts/OperatorRewardsCollector.sol

14:     Initializable,

```

```solidity
File: contracts/Penalty.sol

14: contract Penalty is IPenalty, Initializable, AccessControlUpgradeable, ReentrancyGuardUpgradeable {

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

24:     Initializable,

```

```solidity
File: contracts/PermissionedPool.sol

21: contract PermissionedPool is IStaderPoolBase, Initializable, AccessControlUpgradeable, ReentrancyGuardUpgradeable {

```

```solidity
File: contracts/PermissionlessPool.sol

19: contract PermissionlessPool is IStaderPoolBase, Initializable, AccessControlUpgradeable, ReentrancyGuardUpgradeable {

```

```solidity
File: contracts/PoolSelector.sol

14: contract PoolSelector is IPoolSelector, Initializable, AccessControlUpgradeable {

```

```solidity
File: contracts/PoolUtils.sol

12: contract PoolUtils is IPoolUtils, Initializable, AccessControlUpgradeable {

```

```solidity
File: contracts/SDCollateral.sol

16: contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable, ReentrancyGuardUpgradeable {

```

```solidity
File: contracts/SocializingPool.sol

19:     Initializable,

```

```solidity
File: contracts/StaderConfig.sol

10: contract StaderConfig is IStaderConfig, Initializable, AccessControlUpgradeable {

```

```solidity
File: contracts/StaderInsuranceFund.sol

15:     Initializable,

```

```solidity
File: contracts/UserWithdrawalManager.sol

20:     Initializable,

```

```solidity
File: contracts/factory/VaultFactory.sol

12: contract VaultFactory is IVaultFactory, Initializable, AccessControlUpgradeable {

```

### [NC-6] USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’

#### Description:

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace.

Instead, the Solidity docs recommend specifying imported symbols explicitly.

https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

[code4arena example](https://code4rena.com/reports/2022-10-holograph/#19-non-usage-of-specific-imports)

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

4: import './library/UtilLib.sol';

6: import '../contracts/interfaces/SDCollateral/IAuction.sol';

7: import '../contracts/interfaces/IStaderStakePoolManager.sol';

9: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: contracts/ETHx.sol

3: import './library/UtilLib.sol';

5: import './interfaces/IStaderConfig.sol';

7: import '@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol';

8: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

```

```solidity
File: contracts/NodeELRewardVault.sol

4: import './library/UtilLib.sol';

6: import './interfaces/IPoolUtils.sol';

7: import './interfaces/IVaultProxy.sol';

8: import './interfaces/INodeRegistry.sol';

9: import './interfaces/INodeELRewardVault.sol';

10: import './interfaces/IStaderStakePoolManager.sol';

11: import './interfaces/IOperatorRewardsCollector.sol';

```

```solidity
File: contracts/OperatorRewardsCollector.sol

4: import './library/UtilLib.sol';

6: import './interfaces/IOperatorRewardsCollector.sol';

7: import './interfaces/IStaderConfig.sol';

9: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

```

```solidity
File: contracts/Penalty.sol

4: import './library/UtilLib.sol';

6: import './interfaces/IPenalty.sol';

7: import './interfaces/IRatedV1.sol';

8: import './interfaces/IStaderOracle.sol';

9: import './interfaces/IStaderConfig.sol';

11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

4: import './library/UtilLib.sol';

6: import './library/ValidatorStatus.sol';

7: import './interfaces/IStaderConfig.sol';

8: import './interfaces/IVaultFactory.sol';

9: import './interfaces/IPoolUtils.sol';

10: import './interfaces/INodeRegistry.sol';

11: import './interfaces/IPermissionedPool.sol';

12: import './interfaces/IValidatorWithdrawalVault.sol';

13: import './interfaces/SDCollateral/ISDCollateral.sol';

14: import './interfaces/IPermissionedNodeRegistry.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

17: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: contracts/PermissionedPool.sol

4: import './library/UtilLib.sol';

6: import './library/ValidatorStatus.sol';

8: import './interfaces/IStaderConfig.sol';

9: import './interfaces/IVaultFactory.sol';

10: import './interfaces/INodeRegistry.sol';

11: import './interfaces/IStaderPoolBase.sol';

12: import './interfaces/IDepositContract.sol';

13: import './interfaces/IStaderInsuranceFund.sol';

14: import './interfaces/IStaderStakePoolManager.sol';

15: import './interfaces/IPermissionedNodeRegistry.sol';

17: import '@openzeppelin/contracts/utils/math/Math.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

4: import './library/UtilLib.sol';

6: import './library/ValidatorStatus.sol';

7: import './interfaces/IStaderConfig.sol';

8: import './interfaces/IVaultFactory.sol';

9: import './interfaces/IPoolUtils.sol';

10: import './interfaces/INodeRegistry.sol';

11: import './interfaces/IPermissionlessPool.sol';

12: import './interfaces/INodeELRewardVault.sol';

13: import './interfaces/IStaderInsuranceFund.sol';

14: import './interfaces/IValidatorWithdrawalVault.sol';

15: import './interfaces/SDCollateral/ISDCollateral.sol';

16: import './interfaces/IPermissionlessNodeRegistry.sol';

17: import './interfaces/IOperatorRewardsCollector.sol';

19: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

20: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

21: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: contracts/PermissionlessPool.sol

4: import './library/UtilLib.sol';

5: import './library/ValidatorStatus.sol';

7: import './interfaces/IStaderConfig.sol';

8: import './interfaces/IVaultFactory.sol';

9: import './interfaces/INodeRegistry.sol';

10: import './interfaces/IStaderPoolBase.sol';

11: import './interfaces/IDepositContract.sol';

12: import './interfaces/IStaderStakePoolManager.sol';

13: import './interfaces/IPermissionlessNodeRegistry.sol';

15: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

17: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: contracts/PoolSelector.sol

4: import './library/UtilLib.sol';

6: import './interfaces/IStaderConfig.sol';

7: import './interfaces/IPoolSelector.sol';

8: import './interfaces/IPoolUtils.sol';

10: import '@openzeppelin/contracts/utils/math/Math.sol';

11: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

12: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

```

```solidity
File: contracts/PoolUtils.sol

4: import './library/UtilLib.sol';

6: import './interfaces/IPoolUtils.sol';

7: import './interfaces/IStaderPoolBase.sol';

8: import './interfaces/IStaderConfig.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

```

```solidity
File: contracts/SDCollateral.sol

4: import './library/UtilLib.sol';

6: import '../contracts/interfaces/IPoolUtils.sol';

7: import '../contracts/interfaces/SDCollateral/ISDCollateral.sol';

8: import '../contracts/interfaces/SDCollateral/IAuction.sol';

9: import '../contracts/interfaces/IStaderOracle.sol';

11: import '@openzeppelin/contracts/utils/math/Math.sol';

12: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

14: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

```

```solidity
File: contracts/SocializingPool.sol

5: import './library/UtilLib.sol';

7: import './interfaces/ISocializingPool.sol';

8: import './interfaces/IStaderStakePoolManager.sol';

9: import './interfaces/IPermissionlessNodeRegistry.sol';

11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/utils/cryptography/MerkleProofUpgradeable.sol';

15: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

```

```solidity
File: contracts/StaderConfig.sol

4: import './library/UtilLib.sol';

6: import './interfaces/IStaderConfig.sol';

8: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

```

```solidity
File: contracts/StaderInsuranceFund.sol

4: import './library/UtilLib.sol';

6: import './interfaces/IStaderConfig.sol';

7: import './interfaces/IPermissionedPool.sol';

8: import './interfaces/IStaderInsuranceFund.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: contracts/StaderOracle.sol

4: import './library/UtilLib.sol';

6: import './interfaces/IPoolUtils.sol';

7: import './interfaces/IStaderOracle.sol';

8: import './interfaces/ISocializingPool.sol';

9: import './interfaces/INodeRegistry.sol';

10: import './interfaces/IStaderStakePoolManager.sol';

12: import '@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol';

13: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

15: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: contracts/StaderStakePoolsManager.sol

5: import './library/UtilLib.sol';

7: import './ETHx.sol';

8: import './interfaces/IPoolUtils.sol';

9: import './interfaces/IPoolSelector.sol';

10: import './interfaces/IStaderConfig.sol';

11: import './interfaces/IStaderOracle.sol';

12: import './interfaces/IStaderPoolBase.sol';

13: import './interfaces/IUserWithdrawalManager.sol';

14: import './interfaces/IStaderStakePoolManager.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

17: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

20: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: contracts/UserWithdrawalManager.sol

4: import './library/UtilLib.sol';

6: import './ETHx.sol';

7: import './interfaces/IStaderConfig.sol';

8: import './interfaces/IStaderOracle.sol';

9: import './interfaces/IStaderStakePoolManager.sol';

10: import './interfaces/IUserWithdrawalManager.sol';

12: import '@openzeppelin/contracts/utils/math/Math.sol';

13: import '@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

15: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

16: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

4: import './library/UtilLib.sol';

5: import './library/ValidatorStatus.sol';

7: import './VaultProxy.sol';

8: import './interfaces/IPenalty.sol';

9: import './interfaces/IPoolUtils.sol';

10: import './interfaces/INodeRegistry.sol';

11: import './interfaces/IStaderStakePoolManager.sol';

12: import './interfaces/IValidatorWithdrawalVault.sol';

13: import './interfaces/SDCollateral/ISDCollateral.sol';

14: import './interfaces/IOperatorRewardsCollector.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

```

```solidity
File: contracts/VaultProxy.sol

4: import './library/UtilLib.sol';

5: import './interfaces/IVaultProxy.sol';

```

```solidity
File: contracts/factory/VaultFactory.sol

4: import '../library/UtilLib.sol';

5: import '../VaultProxy.sol';

6: import '../interfaces/IVaultFactory.sol';

7: import '../interfaces/IStaderConfig.sol';

9: import '@openzeppelin/contracts-upgradeable/proxy/ClonesUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

```

```solidity
File: contracts/library/UtilLib.sol

4: import '../interfaces/IStaderConfig.sol';

5: import '../interfaces/INodeRegistry.sol';

6: import '../interfaces/IPoolUtils.sol';

7: import '../interfaces/IVaultProxy.sol';

```

#### Recommended Mitigation Steps:

`import {contract1 , contract2} from "filename.sol";` OR Use specific imports syntax per solidity docs recommendation.

### [NC-7] MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL

#### Description:

If someone wants to extend via inheritance, it might make more sense that the overridden initialize(...) function calls the internal {...}\_init function, not the parent public initialize(...) function.

External instead of public would give more sense of the initialize(...) functions to behave like a constructor (only called on deployment, so should only be called externally).

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

#### **Proof Of Concept**

```solidity
File: contracts/ETHx.sol

29:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionedPool.sol

40:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionlessPool.sol

38:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PoolUtils.sol

25:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/SDCollateral.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/SocializingPool.sol

39:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderInsuranceFund.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderOracle.sol

62:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderStakePoolsManager.sol

50:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/UserWithdrawalManager.sol

54:     function initialize(address _admin, address _staderConfig) public initializer {

```

### [NC-8] LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION

#### Description:

OUse (e.g. 1e6) rather than decimal literals (e.g. 100000), for better code readability.

#### **Proof Of Concept**

```solidity
File: contracts/PoolSelector.sol

21:     uint256 public constant POOL_WEIGHTS_SUM = 10000;

```

```solidity
File: contracts/StaderConfig.sol

92:         setConstant(TOTAL_FEE, 10000);

96:         setVariable(MAX_DEPOSIT_AMOUNT, 10000 ether);

98:         setVariable(MAX_WITHDRAW_AMOUNT, 10000 ether);

```

```solidity
File: contracts/StaderOracle.sol

27:     uint256 public constant ER_CHANGE_MAX_BPS = 10000;

```

### [NC-9] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/ETHx.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/NodeELRewardVault.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/OperatorRewardsCollector.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/Penalty.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionedPool.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionlessPool.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PoolSelector.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PoolUtils.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/SDCollateral.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/SocializingPool.sol

3: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderConfig.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderInsuranceFund.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderOracle.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderStakePoolsManager.sol

3: pragma solidity 0.8.16;

```

```solidity
File: contracts/UserWithdrawalManager.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/VaultProxy.sol

2: pragma solidity ^0.8.16;

```

```solidity
File: contracts/factory/VaultFactory.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/library/UtilLib.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/library/ValidatorStatus.sol

2: pragma solidity 0.8.16;

```

### [NC-10] NO SAME VALUE INPUT CONTROL

#### Description:

IR HAS TO BE IN INPUT like address oracle should be inputAdd code like this; `if (oracle == _oracle revert ADDRESS_SAME();`

[code4arena example](https://code4rena.com/reports/2022-11-non-fungible#n-02-no-same-value-input-control)

[code4arena example](https://code4rena.com/reports/2022-11-redactedcartel#n-12-no-same-value-input-control)

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

147:         duration = _duration;

153:         bidIncrement = _bidIncrement;

```

```solidity
File: contracts/Penalty.sol

43:         ratedOracleAddress = _ratedOracleAddress;

64:         mevTheftPenaltyPerStrike = _mevTheftPenaltyPerStrike;

71:         missedAttestationPenaltyPerStrike = _missedAttestationPenaltyPerStrike;

78:         validatorExitPenaltyThreshold = _validatorExitPenaltyThreshold;

86:         ratedOracleAddress = _ratedOracleAddress;

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

429:         inputKeyCountLimit = _inputKeyCountLimit;

440:         verifiedKeyBatchSize = _verifiedKeysBatchSize;

451:         maxOperatorId = _maxOperatorId;

```

```solidity
File: contracts/PermissionedPool.sol

241:         protocolFee = _protocolFee;

242:         operatorFee = _operatorFee;

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

327:         inputKeyCountLimit = _inputKeyCountLimit;

338:         maxNonTerminalKeyPerOperator = _maxNonTerminalKeyPerOperator;

349:         verifiedKeyBatchSize = _verifiedKeysBatchSize;

```

```solidity
File: contracts/PermissionlessPool.sol

71:         protocolFee = _protocolFee;

72:         operatorFee = _operatorFee;

```

```solidity
File: contracts/PoolSelector.sol

142:         poolAllocationMaxSize = _poolAllocationMaxSize;

```

```solidity
File: contracts/StaderOracle.sol

375:             validatorStats = _validatorStats;

```

```solidity
File: contracts/StaderStakePoolsManager.sol

111:         excessETHDepositCoolDown = _excessETHDepositCoolDown;

```

```solidity
File: contracts/UserWithdrawalManager.sol

79:         finalizationBatchLimit = _finalizationBatchLimit;

```

```solidity
File: contracts/VaultProxy.sol

30:         isValidatorWithdrawalVault = _isValidatorWithdrawalVault;

32:         poolId = _poolId;

33:         id = _id;

70:         owner = _owner;

```

### [NC-11] Add parameter to event-emit

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible.

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

43:         emit UpdatedStaderConfig(_staderConfig);

44:         emit AuctionDurationUpdated(duration);

45:         emit BidIncrementUpdated(bidIncrement);

141:         emit UpdatedStaderConfig(_staderConfig);

148:         emit AuctionDurationUpdated(duration);

154:         emit BidIncrementUpdated(_bidIncrement);

```

```solidity
File: contracts/ETHx.sol

39:         emit UpdatedStaderConfig(_staderConfig);

88:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/OperatorRewardsCollector.sol

37:         emit UpdatedStaderConfig(_staderConfig);

59:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/Penalty.sol

49:         emit UpdatedPenaltyOracleAddress(_ratedOracleAddress);

65:         emit UpdatedMEVTheftPenaltyPerStrike(_mevTheftPenaltyPerStrike);

72:         emit UpdatedMissedAttestationPenaltyPerStrike(_missedAttestationPenaltyPerStrike);

79:         emit UpdatedValidatorExitPenaltyThreshold(_validatorExitPenaltyThreshold);

87:         emit UpdatedPenaltyOracleAddress(_ratedOracleAddress);

94:         emit UpdatedStaderConfig(_staderConfig);

114:                 emit ForceExitValidator(_pubkey[i]);

147:         emit ValidatorMarkedAsSettled(pubkey);

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

95:             emit OperatorWhitelisted(operator);

392:         emit MarkedValidatorStatusAsPreDeposit(_pubkey);

419:         emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);

430:         emit UpdatedInputKeyCountLimit(inputKeyCountLimit);

441:         emit UpdatedVerifiedKeyBatchSize(_verifiedKeysBatchSize);

452:         emit MaxOperatorIdLimitChanged(_maxOperatorId);

462:         emit UpdatedStaderConfig(_staderConfig);

478:         emit IncreasedTotalActiveValidatorCount(totalActiveValidatorCount);

749:         emit DecreasedTotalActiveValidatorCount(totalActiveValidatorCount);

765:         emit OperatorDeactivated(_operatorId);

771:         emit OperatorActivated(_operatorId);

```

```solidity
File: contracts/PermissionedPool.sol

63:         emit ReceivedInsuranceFund(msg.value);

82:         emit TransferredETHToSSPMForDefectiveKeys(amountToSendToPoolManager);

251:         emit UpdatedStaderConfig(_staderConfig);

317:         emit ValidatorPreDepositedOnBeaconChain(pubkey);

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

328:         emit UpdatedInputKeyCountLimit(inputKeyCountLimit);

339:         emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);

350:         emit UpdatedVerifiedKeyBatchSize(_verifiedKeysBatchSize);

357:         emit UpdatedStaderConfig(_staderConfig);

384:         emit IncreasedTotalActiveValidatorCount(totalActiveValidatorCount);

395:         emit TransferredCollateralToPool(_amount);

709:         emit DecreasedTotalActiveValidatorCount(totalActiveValidatorCount);

```

```solidity
File: contracts/PermissionlessPool.sol

62:         emit ReceivedCollateralETH(msg.value);

115:             emit ValidatorPreDepositedOnBeaconChain(_pubkey[i]);

213:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/PoolSelector.sol

143:         emit UpdatedPoolAllocationMaxSize(_poolAllocationMaxSize);

150:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/PoolUtils.sol

76:             emit ExitValidator(_pubkeys[i]);

87:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/SDCollateral.sol

36:         emit UpdatedStaderConfig(_staderConfig);

116:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/SocializingPool.sol

103:         emit UserETHRewardsTransferred(_rewardsData.userETHRewards);

104:         emit ProtocolETHRewardsTransferred(_rewardsData.protocolETHRewards);

182:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/StaderInsuranceFund.sol

37:         emit ReceivedInsuranceFund(msg.value);

52:         emit FundWithdrawn(_amount);

72:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/StaderOracle.sol

77:         emit UpdatedStaderConfig(_staderConfig);

90:         emit TrustedNodeAdded(_nodeAddress);

103:         emit TrustedNodeRemoved(_nodeAddress);

500:         emit SafeModeEnabled();

505:         emit SafeModeDisabled();

512:         emit UpdatedStaderConfig(_staderConfig);

526:         emit ERDataSourceToggled(isPORFeedBasedERData);

536:         emit UpdatedERChangeLimit(erChangeLimit);

568:         emit UpdateFrequencyUpdated(_updateFrequency);

```

```solidity
File: contracts/StaderStakePoolsManager.sol

74:         emit ExecutionLayerRewardsReceived(msg.value);

79:         emit WithdrawVaultUserShareReceived(msg.value);

83:         emit AuctionedEthReceived(msg.value);

91:         emit ReceivedExcessEthFromPool(_poolId);

106:         emit TransferredETHToUserWithdrawManager(_amount);

112:         emit UpdatedExcessETHDepositCoolDown(_excessETHDepositCoolDown);

119:         emit UpdatedStaderConfig(_staderConfig);

```

```solidity
File: contracts/UserWithdrawalManager.sol

69:         emit ReceivedETH(msg.value);

80:         emit UpdatedFinalizationBatchLimit(_finalizationBatchLimit);

87:         emit UpdatedStaderConfig(_staderConfig);

157:             emit FinalizedWithdrawRequest(requestId);

```

```solidity
File: contracts/VaultProxy.sol

60:         emit UpdatedStaderConfig(_staderConfig);

71:         emit UpdatedOwner(owner);

```

```solidity
File: contracts/factory/VaultFactory.sol

44:         emit WithdrawVaultCreated(withdrawVaultAddress);

58:         emit NodeELRewardVaultCreated(nodeELRewardVaultAddress);

89:         emit UpdatedStaderConfig(_staderConfig);

96:         emit UpdatedVaultProxyImplementation(vaultProxyImplementation);

```

### [NC-12] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC

#### Description:

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

#### Exploit Scenario:

A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### **Proof Of Concept**

```solidity
File: hardhat.config.ts

  solidity: {
    compilers: [
      {
        version: '0.8.16',
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
```

#### Recommended Mitigation Steps

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug.

Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

### [NC-13] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/ETHx.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/NodeELRewardVault.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/OperatorRewardsCollector.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/Penalty.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionedPool.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionlessPool.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PoolSelector.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PoolUtils.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/SDCollateral.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/SocializingPool.sol

3: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderConfig.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderInsuranceFund.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderOracle.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderStakePoolsManager.sol

3: pragma solidity 0.8.16;

```

```solidity
File: contracts/UserWithdrawalManager.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/VaultProxy.sol

2: pragma solidity ^0.8.16;

```

```solidity
File: contracts/factory/VaultFactory.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/library/UtilLib.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/library/ValidatorStatus.sol

2: pragma solidity 0.8.16;

```

### [NC-14] Consider renaming this function getVaultFactory() to getVaultFactoryAddress()

#### Description:

Consider renaming this function getVaultFactory() to getVaultFactoryAddress() simply because its purpose seems to be to return an address, not a VaultFactory like the public getter.

#### **Proof Of Concept**

```solidity
File: contracts/PermissionedNodeRegistry.sol

153:         address vaultFactory = staderConfig.getVaultFactory();

```

```solidity
File: contracts/PermissionedPool.sol

93:         address vaultFactory = staderConfig.getVaultFactory();

132:         address vaultFactory = staderConfig.getVaultFactory();

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

106:         address nodeELRewardVault = IVaultFactory(staderConfig.getVaultFactory()).deployNodeELRewardVault(

138:         address vaultFactory = staderConfig.getVaultFactory();

```

```solidity
File: contracts/PermissionlessPool.sol

92:         address vaultFactory = staderConfig.getVaultFactory();

134:         address vaultFactoryAddress = staderConfig.getVaultFactory();

```

```solidity
File: contracts/StaderConfig.sol

387:     function getVaultFactory() external view override returns (address) {

```

### [NC-15] Functions not used internally could be marked external

#### **Proof Of Concept**

```solidity
File: contracts/ETHx.sol

29:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionedPool.sol

40:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

440:     function getTotalQueuedValidatorCount() public view override returns (uint256) {

```

```solidity
File: contracts/PermissionlessPool.sol

38:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PoolUtils.sol

25:     function initialize(address _admin, address _staderConfig) public initializer {

136:     function getSocializingPoolAddress(uint8 _poolId)

147:     function getOperatorTotalNonTerminalKeys(

```

```solidity
File: contracts/SDCollateral.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

205:     function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {

```

```solidity
File: contracts/SocializingPool.sol

39:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderConfig.sol

504:     function onlyManagerRole(address account) public view override returns (bool) {

508:     function onlyOperatorRole(address account) public view override returns (bool) {

```

```solidity
File: contracts/StaderInsuranceFund.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderOracle.sol

62:     function initialize(address _admin, address _staderConfig) public initializer {

571:     function getERReportableBlock() public view override returns (uint256) {

582:     function getSDPriceReportableBlock() public view override returns (uint256) {

586:     function getValidatorStatsReportableBlock() public view override returns (uint256) {

590:     function getWithdrawnValidatorReportableBlock() public view override returns (uint256) {

```

```solidity
File: contracts/StaderStakePoolsManager.sol

50:     function initialize(address _admin, address _staderConfig) public initializer {

140:     function convertToShares(uint256 _assets) public view override returns (uint256) {

145:     function convertToAssets(uint256 _shares) public view override returns (uint256) {

164:     function previewWithdraw(uint256 _shares) public view override returns (uint256) {

169:     function deposit(address _receiver) public payable override whenNotPaused returns (uint256) {

```

```solidity
File: contracts/UserWithdrawalManager.sol

54:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/factory/VaultFactory.sol

34:     function deployWithdrawVault(

48:     function deployNodeELRewardVault(uint8 _poolId, uint256 _operatorId)

62:     function computeWithdrawVaultAddress(

71:     function computeNodeELRewardVaultAddress(uint8 _poolId, uint256 _operatorId)

```

## Low Issues

|      | Issue                                                                      |
| ---- | :------------------------------------------------------------------------- |
| L-1  | Initializing state-variables in proxy-based upgradeable contracts          |
| L-2  | Avoid `transfer()`/`send()` as reentrancy mitigations                      |
| L-3  | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE                             |
| L-4  | Empty Function Body - Consider commenting why                              |
| L-5  | INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY                             |
| L-6  | LACK OF INPUT VALIDATION                                                   |
| L-7  | MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS |
| L-8  | Unify return criteria                                                      |
| L-9  | Timestamp Dependence                                                       |
| L-10 | ACCOUNT EXISTENCE CHECK FOR LOW-LEVEL CALLS                                |
| L-11 | Unsafe ERC20 operation(s)                                                  |
| L-12 | UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT                    |
| L-13 | USE `_SAFEMINT` INSTEAD OF `_MINT`                                         |
| L-14 | USE `SAFETRANSFER` INSTEAD OF `TRANSFER`                                   |
| L-15 | Void constructor                                                           |
| L-16 | NOT USING THE LATEST VERSION OF OPENZEPPELIN FROM DEPENDENCIES             |

### [L-1] Initializing state-variables in proxy-based upgradeable contracts

#### Description:

This should be done in initializer functions and not as part of the state variable declarations in which case they won’t be set.

https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#avoid-initial-values-in-field-declarations

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

29:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: contracts/ETHx.sol

29:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/OperatorRewardsCollector.sol

27:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: contracts/Penalty.sol

31:     function initialize(

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionedPool.sol

40:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionlessPool.sol

38:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PoolSelector.sol

36:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: contracts/PoolUtils.sol

25:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/SDCollateral.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/SocializingPool.sol

39:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderConfig.sol

85:     function initialize(address _admin, address _ethDepositContract) external initializer {

```

```solidity
File: contracts/StaderInsuranceFund.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderOracle.sol

62:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderStakePoolsManager.sol

50:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/UserWithdrawalManager.sol

54:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/factory/VaultFactory.sol

23:     function initialize(address _admin, address _staderConfig) external initializer {

```

### [L-2] Avoid `transfer()`/`send()` as reentrancy mitigations

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

https://swcregistry.io/docs/SWC-134

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

87:         if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {

114:         if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {

```

```solidity
File: contracts/SDCollateral.sol

68:         if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {

```

```solidity
File: contracts/SocializingPool.sol

129:             if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {

```

#### Recommended Mitigation Steps:

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60

### [L-3] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE

#### Description:

The critical procedures should be two step process.

#### **Proof Of Concept**

```solidity
File: contracts/Penalty.sol

53:     function setAdditionalPenaltyAmount(bytes calldata _pubkey, uint256 _amount) external override {

```

```solidity
File: contracts/PermissionedPool.sol

236:     function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {

```

```solidity
File: contracts/PermissionlessPool.sol

66:     function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {

```

```solidity
File: contracts/StaderConfig.sol

471:     function setConstant(bytes32 key, uint256 val) internal {

476:     function setVariable(bytes32 key, uint256 val) internal {

481:     function setAccount(bytes32 key, address val) internal {

487:     function setContract(bytes32 key, address val) internal {

493:     function setToken(bytes32 key, address val) internal {

```

```solidity
File: contracts/StaderOracle.sol

515:     function setERUpdateFrequency(uint256 _updateFrequency) external override {

539:     function setSDPriceUpdateFrequency(uint256 _updateFrequency) external override {

544:     function setValidatorStatsUpdateFrequency(uint256 _updateFrequency) external override {

549:     function setWithdrawnValidatorsUpdateFrequency(uint256 _updateFrequency) external override {

554:     function setMissedAttestationPenaltyUpdateFrequency(uint256 _updateFrequency) external override {

559:     function setUpdateFrequency(bytes32 _key, uint256 _updateFrequency) internal {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-4] Empty Function Body - Consider commenting why

#### Description:

Code contains empty block

Empty blocks should be removed or emit something - The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be `abstract` and the function signatures be added without any default implementation. If the block is an empty `if`-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (`if(x){}else if(y){...}else{...}` => `if(!x){if(y){...}else{...}}`). Empty `receive()`/`fallback()` payable functions that are not used, can be removed to save deployment gas.

#### **Proof Of Concept**

```solidity
File: contracts/NodeELRewardVault.sol

14:     constructor() {}

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

23:     constructor() {}

```

```solidity
File: contracts/VaultProxy.sol

17:     constructor() {}

```

#### Recommended Mitigation Steps:

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

### [L-5] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY

#### Description:

`initialize()` function can be called anybody when the contract is not initialized.

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

29:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: contracts/ETHx.sol

29:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/OperatorRewardsCollector.sol

27:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: contracts/Penalty.sol

35:     ) external initializer {

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionedPool.sol

40:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionlessPool.sol

38:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PoolSelector.sol

36:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: contracts/PoolUtils.sol

25:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/SDCollateral.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/SocializingPool.sol

39:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderConfig.sol

85:     function initialize(address _admin, address _ethDepositContract) external initializer {

```

```solidity
File: contracts/StaderInsuranceFund.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderOracle.sol

62:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderStakePoolsManager.sol

50:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/UserWithdrawalManager.sol

54:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/factory/VaultFactory.sol

23:     function initialize(address _admin, address _staderConfig) external initializer {

```

### [L-6] LACK OF INPUT VALIDATION

#### Description:

For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.

#### **Proof Of Concept**

```solidity
File: contracts/ETHx.sol

47:     function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) whenNotPaused {

56:     function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {

```

### [L-17] MINE - LOSS OF PRECISION DUE TO ROUNDING or Divide before multiply: Performing multiplication before division is generally better to avoid loss of precision because Solidity integer division might truncate.

#### Description:

LOOKING FOR MULTIPLICATION AND THEN DIVISION LIKE HERE

u`int256 fee = (price * fees[i].rate) / INVERSE_BASIS_POINT;`

`((amount * IERC20(to).balanceOf(address(this))) / IERC20(from).balanceOf(address(this)))`

[code4arena example](https://code4rena.com/reports/2022-11-non-fungible#l-07-loss-of-precision-due-to-rounding)

[code4arena example](https://code4rena.com/reports/2022-12-prepo#l-09-loss-of-precision-due-to-rounding)

[code4arena example](https://code4rena.com/reports/2022-12-caviar/#l-03-loss-of-precision-due-to-rounding)

[code4arena example](https://code4rena.com/reports/2022-11-redactedcartel#l-08-loss-of-precision-due-to-rounding)

[code4arena example](https://code4rena.com/reports/2022-11-stakehouse#l-07-loss-of-precision-due-to-rounding)

Divide before multiply: Performing multiplication before division is generally better to avoid loss of precision because Solidity integer division might truncate.

https://github.com/crytic/slither/wiki/Detector-Documentation#divide-before-multiply

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.`

```solidity
File: contracts/PoolSelector.sol

61:         uint256 poolTotalTarget = (poolWeights[_poolId] * totalValidatorsRequired) / POOL_WEIGHTS_SUM;

```

```solidity
File: contracts/PoolUtils.sol

263:         protocolShare = (protocolFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();

265:         operatorShare = (_totalRewards * collateralETH) / TOTAL_STAKED_ETH;

266:         operatorShare += (operatorFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();

```

```solidity
File: contracts/SDCollateral.sol

207:         return (_sdAmount * sdPriceInETH) / staderConfig.getDecimals();

212:         return (_ethAmount * staderConfig.getDecimals()) / sdPriceInETH;

212:         return (_ethAmount * staderConfig.getDecimals()) / sdPriceInETH;

```

```solidity
File: contracts/StaderOracle.sol

603:         return (block.number / updateFrequency) * updateFrequency;

665:             !(newExchangeRate >= (currentExchangeRate * (ER_CHANGE_MAX_BPS - erChangeLimit)) / ER_CHANGE_MAX_BPS &&

666:                 newExchangeRate <= ((currentExchangeRate * (ER_CHANGE_MAX_BPS + erChangeLimit)) / ER_CHANGE_MAX_BPS))

```

```solidity
File: contracts/StaderStakePoolsManager.sol

207:         IStaderPoolBase(poolAddress).stakeUserETHToBeaconChain{value: selectedPoolCapacity * poolDepositSize}();

```

### [L-7] MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS

#### Description:

Most contracts use the delegateCall proxy pattern and hence their implementations require the use of `initialize()` functions instead of constructors. This requires derived contracts to call the corresponding init functions of their inherited base contracts. This is done in most places except a few.

Impact: The inherited base classes do not get initialized which may lead to undefined behavior.

Missing call to `__ReentrancyGuard_init`

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

29:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: contracts/ETHx.sol

29:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/OperatorRewardsCollector.sol

27:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: contracts/Penalty.sol

31:     function initialize(

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionedPool.sol

40:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PermissionlessPool.sol

38:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/PoolSelector.sol

36:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: contracts/PoolUtils.sol

25:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/SDCollateral.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/SocializingPool.sol

39:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderConfig.sol

85:     function initialize(address _admin, address _ethDepositContract) external initializer {

```

```solidity
File: contracts/StaderInsuranceFund.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderOracle.sol

62:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/StaderStakePoolsManager.sol

50:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/UserWithdrawalManager.sol

54:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: contracts/factory/VaultFactory.sol

23:     function initialize(address _admin, address _staderConfig) external initializer {

```

#### Recommended Mitigation Steps:

Add missing calls to init functions of inherited contracts.

### [L-8] Unify return criteria

#### Description:

In contracts, sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: contracts/Penalty.sol

123:     function calculateMEVTheftPenalty(bytes32 _pubkeyRoot) public override returns (uint256) {

132:     function calculateMissedAttestationPenalty(bytes32 _pubkeyRoot) public view override returns (uint256) {

139:     function getAdditionalPenaltyAmount(bytes calldata _pubkey) external view override returns (uint256) {

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

110:         returns (address feeRecipientAddress)

195:         returns (uint256[] memory selectedOperatorCapacity)

466:     function getSocializingPoolStateChangeBlock(uint256 _operatorId) external view returns (uint256) {

486:     function getTotalQueuedValidatorCount() external view override returns (uint256) {

504:     function getTotalActiveValidatorCount() external view override returns (uint256) {

513:     function getOperatorTotalKeys(uint256 _operatorId) public view override returns (uint256 _totalKeys) {

526:     ) public view override returns (uint64) {

546:     function getCollateralETH() external pure override returns (uint256) {

554:     function getOperatorRewardAddress(uint256 _operatorId) external view override returns (address payable) {

588:         returns (Validator[] memory)

624:     ) external view override returns (Validator[] memory) {

646:     function isExistingPubkey(bytes calldata _pubkey) external view override returns (bool) {

651:     function isExistingOperator(address _operAddr) external view override returns (bool) {

680:     function getOperatorQueuedValidatorCount(uint256 _operatorId) internal view returns (uint256 _validatorCount) {

692:     ) internal view returns (uint256 keyCount, uint256 totalKeys) {

720:     function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {

732:     function isActiveValidator(uint256 _validatorId) internal view returns (bool) {

738:     function isNonTerminalValidator(uint256 _validatorId) internal view returns (bool) {

```

```solidity
File: contracts/PermissionedPool.sol

185:     function getTotalQueuedValidatorCount() external view override returns (uint256) {

192:     function getTotalActiveValidatorCount() external view override returns (uint256) {

203:     ) external view override returns (uint256) {

213:     function getSocializingPoolAddress() external view returns (address) {

217:     function getCollateralETH() external view override returns (uint256) {

221:     function getNodeRegistry() external view override returns (address) {

226:     function isExistingPubkey(bytes calldata _pubkey) external view override returns (bool) {

231:     function isExistingOperator(address _operAddr) external view override returns (bool) {

261:     ) external pure returns (bytes32) {

321:     function to_little_endian_64(uint256 _depositAmount) internal pure returns (bytes memory ret) {

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

93:     ) external override whenNotPaused returns (address feeRecipientAddress) {

290:         returns (address feeRecipientAddress)

316:     function getSocializingPoolStateChangeBlock(uint256 _operatorId) external view returns (uint256) {

407:     ) public view override returns (uint64) {

432:     function getOperatorTotalKeys(uint256 _operatorId) public view override returns (uint256 _totalKeys) {

440:     function getTotalQueuedValidatorCount() public view override returns (uint256) {

448:     function getTotalActiveValidatorCount() external view override returns (uint256) {

452:     function getCollateralETH() external pure override returns (uint256) {

460:     function getOperatorRewardAddress(uint256 _operatorId) external view override returns (address payable) {

494:         returns (Validator[] memory)

530:     ) external view override returns (Validator[] memory) {

563:         returns (address[] memory)

589:     function isExistingPubkey(bytes calldata _pubkey) external view override returns (bool) {

594:     function isExistingOperator(address _operAddr) external view override returns (bool) {

648:     ) internal view returns (uint256 keyCount, uint256 totalKeys) {

680:     function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {

691:     function isNonTerminalValidator(uint256 _validatorId) internal view returns (bool) {

701:     function isActiveValidator(uint256 _validatorId) internal view returns (bool) {

```

```solidity
File: contracts/PermissionlessPool.sol

157:     function getSocializingPoolAddress() external view returns (address) {

164:     function getTotalQueuedValidatorCount() external view override returns (uint256) {

171:     function getTotalActiveValidatorCount() external view override returns (uint256) {

182:     ) external view override returns (uint256) {

191:     function getCollateralETH() external view override returns (uint256) {

195:     function getNodeRegistry() external view override returns (address) {

200:     function isExistingPubkey(bytes calldata _pubkey) external view override returns (bool) {

205:     function isExistingOperator(address _operAddr) external view override returns (bool) {

223:     ) external pure returns (bytes32) {

273:     function to_little_endian_64(uint256 _depositAmount) internal pure returns (bytes memory ret) {

```

```solidity
File: contracts/PoolSelector.sol

54:         returns (uint256 selectedPoolCapacity)

79:         returns (uint256[] memory selectedPoolCapacity, uint8[] memory poolIdArray)

```

```solidity
File: contracts/PoolUtils.sol

91:     function getProtocolFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {

96:     function getOperatorFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {

101:     function getTotalActiveValidatorCount() external view override returns (uint256) {

117:         returns (uint256)

129:         returns (uint256)

141:         returns (address)

152:     ) public view override onlyExistingPoolId(_poolId) returns (uint256) {

157:     function getCollateralETH(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {

162:     function getNodeRegistry(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (address) {

166:     function isExistingPubkey(bytes calldata _pubkey) public view override returns (bool) {

177:     function isExistingOperator(address _operAddr) external view override returns (bool) {

188:     function getOperatorPoolId(address _operAddr) external view override returns (uint8) {

199:     function getValidatorPoolId(bytes calldata _pubkey) external view override returns (uint8) {

210:     function getPoolIdArray() external view override returns (uint8[] memory) {

249:         returns (

271:     function isExistingPoolId(uint8 _poolId) public view override returns (bool) {

290:     function getPoolCount() internal view returns (uint256) {

```

```solidity
File: contracts/SDCollateral.sol

144:     function getOperatorWithdrawThreshold(address _operator) public view returns (uint256 operatorWithdrawThreshold) {

159:     ) external view override returns (bool) {

170:         returns (uint256 _minSDToBond)

187:     ) public view override returns (uint256) {

193:     function getRewardEligibleSD(address _operator) external view override returns (uint256 _rewardEligibleSD) {

205:     function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {

210:     function convertETHToSD(uint256 _ethAmount) public view override returns (uint256) {

220:         returns (

```

```solidity
File: contracts/SocializingPool.sol

143:     ) internal returns (uint256 _totalAmountSD, uint256 _totalAmountETH) {

169:     ) public view returns (bool) {

187:     function getCurrentRewardsIndex() public view returns (uint256 index) {

195:         returns (

206:     function getRewardCycleDetails(uint256 _index) public view returns (uint256 _startBlock, uint256 _endBlock) {

```

```solidity
File: contracts/StaderConfig.sol

297:     function getStakedEthPerNode() external view override returns (uint256) {

301:     function getPreDepositSize() external view override returns (uint256) {

305:     function getFullDepositSize() external view override returns (uint256) {

309:     function getDecimals() external view override returns (uint256) {

313:     function getTotalFee() external view override returns (uint256) {

317:     function getOperatorMaxNameLength() external view override returns (uint256) {

323:     function getSocializingPoolCycleDuration() external view override returns (uint256) {

327:     function getSocializingPoolOptInCoolingPeriod() external view override returns (uint256) {

331:     function getRewardsThreshold() external view override returns (uint256) {

335:     function getMinDepositAmount() external view override returns (uint256) {

339:     function getMaxDepositAmount() external view override returns (uint256) {

343:     function getMinWithdrawAmount() external view override returns (uint256) {

347:     function getMaxWithdrawAmount() external view override returns (uint256) {

351:     function getMinBlockDelayToFinalizeWithdrawRequest() external view override returns (uint256) {

355:     function getWithdrawnKeyBatchSize() external view override returns (uint256) {

361:     function getAdmin() external view returns (address) {

365:     function getStaderTreasury() external view override returns (address) {

371:     function getPoolUtils() external view override returns (address) {

375:     function getPoolSelector() external view override returns (address) {

379:     function getSDCollateral() external view override returns (address) {

383:     function getOperatorRewardsCollector() external view override returns (address) {

387:     function getVaultFactory() external view override returns (address) {

391:     function getStaderOracle() external view override returns (address) {

395:     function getAuctionContract() external view override returns (address) {

399:     function getPenaltyContract() external view override returns (address) {

403:     function getPermissionedPool() external view override returns (address) {

407:     function getStakePoolManager() external view override returns (address) {

411:     function getETHDepositContract() external view override returns (address) {

415:     function getPermissionlessPool() external view override returns (address) {

419:     function getUserWithdrawManager() external view override returns (address) {

423:     function getStaderInsuranceFund() external view override returns (address) {

427:     function getPermissionedNodeRegistry() external view override returns (address) {

431:     function getPermissionlessNodeRegistry() external view override returns (address) {

435:     function getPermissionedSocializingPool() external view override returns (address) {

439:     function getPermissionlessSocializingPool() external view override returns (address) {

443:     function getNodeELRewardVaultImplementation() external view override returns (address) {

447:     function getValidatorWithdrawalVaultImplementation() external view override returns (address) {

452:     function getETHBalancePORFeedProxy() external view override returns (address) {

456:     function getETHXSupplyPORFeedProxy() external view override returns (address) {

462:     function getStaderToken() external view override returns (address) {

466:     function getETHxToken() external view returns (address) {

500:     function onlyStaderContract(address _addr, bytes32 _contractName) external view returns (bool) {

504:     function onlyManagerRole(address account) public view override returns (bool) {

508:     function onlyOperatorRole(address account) public view override returns (bool) {

```

```solidity
File: contracts/StaderOracle.sol

312:     function getMedianValue(uint256[] storage dataArray) internal view returns (uint256 _medianValue) {

571:     function getERReportableBlock() public view override returns (uint256) {

575:     function getMerkleRootReportableBlockByPoolId(uint8 _poolId) public view override returns (uint256) {

582:     function getSDPriceReportableBlock() public view override returns (uint256) {

586:     function getValidatorStatsReportableBlock() public view override returns (uint256) {

590:     function getWithdrawnValidatorReportableBlock() public view override returns (uint256) {

594:     function getMissedAttestationPenaltyReportableBlock() public view override returns (uint256) {

598:     function getReportableBlockFor(bytes32 _key) internal view returns (uint256) {

606:     function getCurrentRewardsIndexByPoolId(uint8 _poolId) public view returns (uint256) {

612:     function getValidatorStats() external view override returns (ValidatorStats memory) {

616:     function getExchangeRate() external view override returns (ExchangeRate memory) {

622:         returns (uint8 _submissionCount)

633:     function getSDPriceInETH() external view override returns (uint256) {

640:         returns (

```

```solidity
File: contracts/StaderStakePoolsManager.sol

125:     function getExchangeRate() public view override returns (uint256) {

135:     function totalAssets() public view override returns (uint256) {

140:     function convertToShares(uint256 _assets) public view override returns (uint256) {

145:     function convertToAssets(uint256 _shares) public view override returns (uint256) {

150:     function maxDeposit() public view override returns (uint256) {

154:     function minDeposit() public view override returns (uint256) {

159:     function previewDeposit(uint256 _assets) public view override returns (uint256) {

164:     function previewWithdraw(uint256 _shares) public view override returns (uint256) {

169:     function deposit(address _receiver) public payable override whenNotPaused returns (uint256) {

271:     function _convertToShares(uint256 _assets, Math.Rounding rounding) internal view returns (uint256) {

287:     ) internal pure returns (uint256 shares) {

294:     function _convertToAssets(uint256 _shares, Math.Rounding rounding) internal view returns (uint256) {

308:     ) internal pure returns (uint256) {

328:     function isVaultHealthy() public view override returns (bool) {

```

```solidity
File: contracts/UserWithdrawalManager.sol

95:     function requestWithdraw(uint256 _ethXAmount, address _owner) external override whenNotPaused returns (uint256) {

184:     function getRequestIdsByUser(address _owner) external view override returns (uint256[] memory) {

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

88:         returns (

127:     function getCollateralETH(uint8 _poolId, IStaderConfig _staderConfig) internal view returns (uint256) {

135:     ) internal view returns (address) {

143:     ) internal returns (uint256) {

```

```solidity
File: contracts/VaultProxy.sol

41:     fallback(bytes calldata _input) external payable returns (bytes memory) {

```

```solidity
File: contracts/factory/VaultFactory.sol

39:     ) public override onlyRole(NODE_REGISTRY_CONTRACT) returns (address) {

52:         returns (address)

66:     ) public view override returns (address) {

75:         returns (address)

81:     function getValidatorWithdrawCredential(address _withdrawVault) external pure override returns (bytes memory) {

```

```solidity
File: contracts/library/UtilLib.sol

54:     ) internal view returns (bytes memory) {

70:     ) internal view returns (address) {

99:     ) internal view returns (address) {

111:     ) internal view returns (address) {

121:         returns (address payable)

134:     function getPubkeyRoot(bytes calldata _pubkey) internal pure returns (bytes32) {

146:         returns (bool)

159:     ) internal view returns (uint256) {

```

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

### [L-9] Timestamp Dependence

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

Reference: https://swcregistry.io/docs/SWC-116

Reference: (https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/timestamp-dependence.md)

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

49:         lots[nextLot].startBlock = block.number;

50:         lots[nextLot].endBlock = block.number + duration;

67:         if (block.number > lotItem.endBlock) revert AuctionEnded();

82:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

97:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

108:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

122:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

323:             validatorRegistry[validatorId].withdrawnBlock = block.number;

378:         validatorRegistry[_validatorId].depositBlock = block.number;

380:         emit UpdatedValidatorDepositBlock(_validatorId, block.number);

664:         socializingPoolStateChangeBlock[nextOperatorId] = block.number;

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

253:             validatorRegistry[validatorId].withdrawnBlock = block.number;

281:         validatorRegistry[_validatorId].depositBlock = block.number;

283:         emit UpdatedValidatorDepositBlock(_validatorId, block.number);

298:             block.number <

311:         socializingPoolStateChangeBlock[operatorId] = block.number;

312:         emit UpdatedSocializingPoolState(operatorId, _optInForSocializingPool, block.number);

611:         socializingPoolStateChangeBlock[nextOperatorId] = block.number;

```

```solidity
File: contracts/SocializingPool.sol

48:         initialBlock = block.number;

```

```solidity
File: contracts/StaderOracle.sol

118:         if (_exchangeRate.reportingBlockNumber >= block.number) {

144:             block.timestamp

188:             erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY > block.number

209:         if (_rewardsData.reportingBlockNumber >= block.number) {

250:             block.number

265:                 block.number

271:         if (_sdPriceData.reportingBlockNumber >= block.number) {

287:         emit SDPriceSubmitted(msg.sender, _sdPriceData.sdPriceInETH, _sdPriceData.reportingBlockNumber, block.number);

296:             emit SDPriceUpdated(_sdPriceData.sdPriceInETH, _sdPriceData.reportingBlockNumber, block.number);

325:         if (_validatorStats.reportingBlockNumber >= block.number) {

368:             block.timestamp

386:                 block.timestamp

400:         if (_withdrawnValidators.reportingBlockNumber >= block.number) {

428:             block.timestamp

443:                 block.timestamp

456:         if (_mapd.reportingBlockNumber >= block.number) {

477:             block.number,

492:             emit MissedAttestationPenaltyUpdated(_mapd.index, block.number, _mapd.sortedPubkeys);

603:         return (block.number / updateFrequency) * updateFrequency;

650:         return (uint256(totalETHBalanceInInt), uint256(totalETHXSupplyInInt), block.number);

669:             erInspectionModeStartBlock = block.number;

673:             emit ERInspectionModeActivated(erInspectionMode, block.timestamp);

693:             block.timestamp

```

```solidity
File: contracts/StaderStakePoolsManager.sol

56:         lastExcessETHDepositBlock = block.number;

216:         if (block.number < lastExcessETHDepositBlock + excessETHDepositCoolDown) {

241:             lastExcessETHDepositBlock = block.number;

```

```solidity
File: contracts/UserWithdrawalManager.sol

106:         userWithdrawRequests[nextRequestId] = UserWithdrawInfo(payable(_owner), _ethXAmount, assets, 0, block.number);

140:                     block.number)

```

### [L-10] ACCOUNT EXISTENCE CHECK FOR LOW-LEVEL CALLS

#### Description:

Low-level calls `call`/`delegatecall`/`staticcall` return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling if needed.

https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-callsn

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

131:         (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');

```

```solidity
File: contracts/SocializingPool.sol

91:         (bool success, ) = payable(staderConfig.getStaderTreasury()).call{value: _rewardsData.protocolETHRewards}('');

121:             (success, ) = payable(operatorRewardsAddr).call{value: totalAmountETH}('');

```

```solidity
File: contracts/StaderInsuranceFund.sol

48:         (bool success, ) = payable(msg.sender).call{value: _amount}('');

```

```solidity
File: contracts/StaderStakePoolsManager.sol

102:         (bool success, ) = payable(staderConfig.getUserWithdrawManager()).call{value: _amount}('');

```

```solidity
File: contracts/UserWithdrawalManager.sol

229:         (bool success, ) = _recipient.call{value: _amount}('');

```

```solidity
File: contracts/VaultProxy.sol

45:         (bool success, bytes memory data) = vaultImplementation.delegatecall(_input);

```

```solidity
File: contracts/library/UtilLib.sol

168:         (bool success, ) = payable(_receiver).call{value: _amount}('');

```

#### Recommended Mitigation Steps:

In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

### [L-11] Unsafe ERC20 operation(s)

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

55:         if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {

87:         if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {

114:         if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {

```

```solidity
File: contracts/SDCollateral.sol

47:         if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {

68:         if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {

106:         IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);

```

```solidity
File: contracts/SocializingPool.sol

129:             if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {

```

### [L-12] UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT

#### Description:

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

The function should call another function, otherwise it should revert

```solidity
File: contracts/NodeELRewardVault.sol

20:     receive() external payable {

```

```solidity
File: contracts/SocializingPool.sol

57:     receive() external payable {

```

```solidity
File: contracts/UserWithdrawalManager.sol

68:     receive() external payable {

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

26:     receive() external payable {

```

### [L-13] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`

```solidity
File: contracts/ETHx.sol

48:         _mint(to, amount);

```

### [L-14] USE `SAFETRANSFER` INSTEAD OF `TRANSFER`

#### Description:

It is good to add a `require()` statement that checks the return value of token transfers or to use something like OpenZeppelin’s `safeTransfer`/`safeTransferFrom` unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)‘s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert.

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

87:         if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {

114:         if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {

```

```solidity
File: contracts/SDCollateral.sol

68:         if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {

```

```solidity
File: contracts/SocializingPool.sol

129:             if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {

```

#### Recommended Mitigation Steps:

Consider using `safeTransfer`/`safeTransferFrom` or `require()` consistently.

### [L-15] Void constructor

#### Description:

Calls to base contract constructors that are unimplemented leads to misplaced assumptions. Check if the constructor is implemented or remove call if not.

https://github.com/crytic/slither/wiki/Detector-Documentation#void-constructor

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

25:     constructor() {

```

```solidity
File: contracts/ETHx.sol

25:     constructor() {

```

```solidity
File: contracts/NodeELRewardVault.sol

14:     constructor() {}

```

```solidity
File: contracts/OperatorRewardsCollector.sol

23:     constructor() {

```

```solidity
File: contracts/Penalty.sol

27:     constructor() {

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

62:     constructor() {

```

```solidity
File: contracts/PermissionedPool.sol

36:     constructor() {

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

62:     constructor() {

```

```solidity
File: contracts/PermissionlessPool.sol

34:     constructor() {

```

```solidity
File: contracts/PoolSelector.sol

26:     constructor() {

```

```solidity
File: contracts/PoolUtils.sol

21:     constructor() {

```

```solidity
File: contracts/SDCollateral.sol

22:     constructor() {

```

```solidity
File: contracts/SocializingPool.sol

35:     constructor() {

```

```solidity
File: contracts/StaderConfig.sol

81:     constructor() {

```

```solidity
File: contracts/StaderInsuranceFund.sol

22:     constructor() {

```

```solidity
File: contracts/StaderOracle.sol

58:     constructor() {

```

```solidity
File: contracts/StaderStakePoolsManager.sol

42:     constructor() {

```

```solidity
File: contracts/UserWithdrawalManager.sol

50:     constructor() {

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

23:     constructor() {}

```

```solidity
File: contracts/VaultProxy.sol

17:     constructor() {}

```

```solidity
File: contracts/factory/VaultFactory.sol

19:     constructor() {

```

### [L-16] NOT USING THE LATEST VERSION OF OPENZEPPELIN FROM DEPENDENCIES

#### Description:

The package.json configuration file says that the project is using 4.7.3 etc. of OpenZeppelin which has a not last update version

#### **Proof Of Concept**

```solidity
File: package.json

  "dependencies": {
    "@openzeppelin/contracts": "^4.7.3",

```

#### Recommended Mitigation Steps:

Use patched versions.
