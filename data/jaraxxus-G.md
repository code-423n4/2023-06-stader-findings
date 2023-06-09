## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Using bools for storage incurs overhead | 12 |
| [GAS-2](#GAS-2) | State variables should be cached in stack variables rather than re-reading them from storage | 5 |
| [GAS-3](#GAS-3) | Use calldata instead of memory for function arguments that do not get mutated | 1 |
| [GAS-4](#GAS-4) | Don't initialize variables with default value | 13 |
| [GAS-5](#GAS-5) | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | 34 |
| [GAS-6](#GAS-6) | Using `private` rather than `public` for constants, saves gas | 62 |
| [GAS-7](#GAS-7) | Use shift Right/Left instead of division/multiplication if possible | 8 |
| [GAS-8](#GAS-8) | Use `storage` instead of `memory` for structs/arrays | 32 |
| [GAS-9](#GAS-9) | Increments can be `unchecked` in for-loops | 15 |
| [GAS-10](#GAS-10) | Use != 0 instead of > 0 for unsigned integer comparison | 13 |
| [GAS-11](#GAS-11) | `internal` functions not called by the contract should be removed | 14 |
### [GAS-1] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (12)*:
```solidity
File: example/PermissionedNodeRegistry.sol

38:     mapping(address => bool) public override permissionList;

```

```solidity
File: example/SocializingPool.sol

17:     mapping(address => mapping(uint256 => bool)) public override claimedRewards;

18:     mapping(uint256 => bool) public handledRewards;

```

```solidity
File: example/StaderOracle.sol

5:     bool public override erInspectionMode;

6:     bool public override isPORFeedBasedERData;

28:     bool public override safeMode;

31:     mapping(address => bool) public override isTrustedNode;

32:     mapping(bytes32 => bool) private nodeSubmissionKeys;

```

```solidity
File: example/ValidatorWithdrawalVault.sol

5:     bool internal vaultSettleStatus;

```

```solidity
File: example/VaultProxy.sol

6:     bool public override vaultSettleStatus;

7:     bool public override isValidatorWithdrawalVault;

8:     bool public override isInitialized;

```

### [GAS-2] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (5)*:
```solidity
File: example/PermissionedNodeRegistry.sol

157:             validatorIdByPubkey[_pubkey[i]] = nextValidatorId;

186:         uint256[] memory remainingOperatorCapacity = new uint256[](nextOperatorId);

```

```solidity
File: example/PermissionlessNodeRegistry.sol

141:             validatorIdByPubkey[_pubkey[i]] = nextValidatorId;

```

```solidity
File: example/StaderOracle.sol

650:         uint256 newExchangeRate = UtilLib.computeExchangeRate(_newTotalETHBalance, _newTotalETHXSupply, staderConfig);

```

```solidity
File: example/UserWithdrawalManager.sol

94:         emit WithdrawRequestReceived(msg.sender, _owner, nextRequestId, _ethXAmount, assets);

```

### [GAS-3] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (1)*:
```solidity
File: example/SDCollateral.sol

112:         string memory _units

```

### [GAS-4] Don't initialize variables with default value

*Instances (13)*:
```solidity
File: example/PermissionedNodeRegistry.sol

75:         for (uint256 i = 0; i < permissionedNosLength; i++) {

581:         uint256 validatorCount = 0;

```

```solidity
File: example/PermissionlessNodeRegistry.sol

485:         uint256 validatorCount = 0;

554:         uint256 optOutOperatorCount = 0;

```

```solidity
File: example/PoolSelector.sol

121:         for (uint256 i = 0; i < poolTargetLength; i++) {

```

```solidity
File: example/PoolUtils.sol

96:         for (uint256 i = 0; i < poolCount; i++) {

160:         for (uint256 i = 0; i < poolCount; i++) {

171:         for (uint256 i = 0; i < poolCount; i++) {

182:         for (uint256 i = 0; i < poolCount; i++) {

193:         for (uint256 i = 0; i < poolCount; i++) {

265:         for (uint256 i = 0; i < poolCount; i++) {

```

```solidity
File: example/SocializingPool.sol

133:         for (uint256 i = 0; i < indexLength; i++) {

```

```solidity
File: example/StaderStakePoolsManager.sol

216:         for (uint256 i = 0; i < poolCount; i++) {

```

### [GAS-5] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
*Saves 5 gas per iteration*

*Instances (34)*:
```solidity
File: example/Auction.sol

49:         nextLot++;

```

```solidity
File: example/PermissionedNodeRegistry.sol

75:         for (uint256 i = 0; i < permissionedNosLength; i++) {

160:             nextValidatorId++;

521:                 totalNonWithdrawnKeyCount++;

582:         for (uint256 i = startIndex; i < endIndex; i++) {

585:                 validatorCount++;

621:         for (uint256 i = startIndex; i < endIndex; i++) {

649:         nextOperatorId++;

650:         totalActiveOperatorCount++;

748:         totalActiveOperatorCount--;

754:         totalActiveOperatorCount++;

```

```solidity
File: example/PermissionedPool.sol

95:                 index++

```

```solidity
File: example/PermissionlessNodeRegistry.sol

144:             nextValidatorId++;

400:                 totalNonWithdrawnKeyCount++;

486:         for (uint256 i = startIndex; i < endIndex; i++) {

489:                 validatorCount++;

525:         for (uint256 i = startIndex; i < endIndex; i++) {

555:         for (uint256 i = startIndex; i < endIndex; i++) {

558:                 optOutOperatorCount++;

594:         nextOperatorId++;

603:         validatorQueueSize++;

```

```solidity
File: example/PoolSelector.sol

121:         for (uint256 i = 0; i < poolTargetLength; i++) {

```

```solidity
File: example/PoolUtils.sol

96:         for (uint256 i = 0; i < poolCount; i++) {

160:         for (uint256 i = 0; i < poolCount; i++) {

171:         for (uint256 i = 0; i < poolCount; i++) {

182:         for (uint256 i = 0; i < poolCount; i++) {

193:         for (uint256 i = 0; i < poolCount; i++) {

265:         for (uint256 i = 0; i < poolCount; i++) {

```

```solidity
File: example/SocializingPool.sol

133:         for (uint256 i = 0; i < indexLength; i++) {

```

```solidity
File: example/StaderOracle.sol

75:         trustedNodesCount++;

88:         trustedNodesCount--;

294:             j--;

```

```solidity
File: example/StaderStakePoolsManager.sol

216:         for (uint256 i = 0; i < poolCount; i++) {

```

```solidity
File: example/UserWithdrawalManager.sol

95:         nextRequestId++;

```

### [GAS-6] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (62)*:
```solidity
File: example/Auction.sol

12:     uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours

```

```solidity
File: example/ETHx.sol

14:     bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');

15:     bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');

```

```solidity
File: example/PermissionedNodeRegistry.sol

15:     uint8 public constant override POOL_ID = 2;

```

```solidity
File: example/PermissionedPool.sol

16:     uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

```

```solidity
File: example/PermissionlessNodeRegistry.sol

12:     uint8 public constant override POOL_ID = 1;

24:     uint256 public constant override FRONT_RUN_PENALTY = 3 ether;

25:     uint256 public constant COLLATERAL_ETH = 4 ether;

```

```solidity
File: example/PermissionlessPool.sol

9:     uint256 public constant DEPOSIT_NODE_BOND = 3 ether;

17:     uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

```

```solidity
File: example/PoolSelector.sol

12:     uint256 public constant POOL_WEIGHTS_SUM = 10000;

```

```solidity
File: example/StaderConfig.sol

6:     bytes32 public constant ETH_PER_NODE = keccak256('ETH_PER_NODE');

8:     bytes32 public constant PRE_DEPOSIT_SIZE = keccak256('PRE_DEPOSIT_SIZE');

10:     bytes32 public constant FULL_DEPOSIT_SIZE = keccak256('FULL_DEPOSIT_SIZE');

12:     bytes32 public constant DECIMALS = keccak256('DECIMALS');

14:     bytes32 public constant TOTAL_FEE = keccak256('TOTAL_FEE');

16:     bytes32 public constant OPERATOR_MAX_NAME_LENGTH = keccak256('OPERATOR_MAX_NAME_LENGTH');

18:     bytes32 public constant SOCIALIZING_POOL_CYCLE_DURATION = keccak256('SOCIALIZING_POOL_CYCLE_DURATION');

19:     bytes32 public constant SOCIALIZING_POOL_OPT_IN_COOLING_PERIOD =

21:     bytes32 public constant REWARD_THRESHOLD = keccak256('REWARD_THRESHOLD');

22:     bytes32 public constant MIN_DEPOSIT_AMOUNT = keccak256('MIN_DEPOSIT_AMOUNT');

23:     bytes32 public constant MAX_DEPOSIT_AMOUNT = keccak256('MAX_DEPOSIT_AMOUNT');

24:     bytes32 public constant MIN_WITHDRAW_AMOUNT = keccak256('MIN_WITHDRAW_AMOUNT');

25:     bytes32 public constant MAX_WITHDRAW_AMOUNT = keccak256('MAX_WITHDRAW_AMOUNT');

27:     bytes32 public constant MIN_BLOCK_DELAY_TO_FINALIZE_WITHDRAW_REQUEST =

29:     bytes32 public constant WITHDRAWN_KEYS_BATCH_SIZE = keccak256('WITHDRAWN_KEYS_BATCH_SIZE');

31:     bytes32 public constant ADMIN = keccak256('ADMIN');

32:     bytes32 public constant STADER_TREASURY = keccak256('STADER_TREASURY');

34:     bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');

35:     bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');

36:     bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');

37:     bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');

38:     bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');

39:     bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');

40:     bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');

41:     bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');

42:     bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');

43:     bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');

44:     bytes32 public constant override ETH_DEPOSIT_CONTRACT = keccak256('ETH_DEPOSIT_CONTRACT');

45:     bytes32 public constant override PERMISSIONLESS_POOL = keccak256('PERMISSIONLESS_POOL');

46:     bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');

47:     bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');

48:     bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');

49:     bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');

50:     bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');

51:     bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');

52:     bytes32 public constant override NODE_EL_REWARD_VAULT_IMPLEMENTATION =

54:     bytes32 public constant override VALIDATOR_WITHDRAWAL_VAULT_IMPLEMENTATION =

58:     bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');

59:     bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');

62:     bytes32 public constant override MANAGER = keccak256('MANAGER');

63:     bytes32 public constant override OPERATOR = keccak256('OPERATOR');

65:     bytes32 public constant SD = keccak256('SD');

66:     bytes32 public constant ETHx = keccak256('ETHx');

```

```solidity
File: example/StaderOracle.sol

13:     uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days

14:     uint256 public constant ER_CHANGE_MAX_BPS = 10000;

16:     uint256 public constant MIN_TRUSTED_NODES = 5;

37:     bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency

38:     bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key

39:     bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key

40:     bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key

41:     bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key

```

### [GAS-7] Use shift Right/Left instead of division/multiplication if possible
Shifting left by N is like multiplying by 2^N and shifting right by N is like dividing by 2^N

*Instances (8)*:
```solidity
File: example/StaderOracle.sol

136:             _exchangeRate.reportingBlockNumber > exchangeRate.reportingBlockNumber

243:             address socializingPool = IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(

302:     }

302:     }

302:     }

360:             _validatorStats.reportingBlockNumber > validatorStats.reportingBlockNumber

420:             _withdrawnValidators.reportingBlockNumber > reportingBlockNumberForWithdrawnValidators

470:             lastReportedMAPDIndex = _mapd.index;

```

### [GAS-8] Use `storage` instead of `memory` for structs/arrays
Using `memory` copies the struct or array in memory. Use `storage` to save the location in storage and have cheaper reads:

*Instances (32)*:
```solidity
File: example/Penalty.sol

116: 

136:         totalPenaltyAmount[pubkey] = 0;

```

```solidity
File: example/PermissionedNodeRegistry.sol

187:         uint256 totalValidatorToDeposit;

581:         uint256 validatorCount = 0;

621:         for (uint256 i = startIndex; i < endIndex; i++) {

718:         return (validator.status == ValidatorStatus.PRE_DEPOSIT || validator.status == ValidatorStatus.DEPOSITED);

724:         return

```

```solidity
File: example/PermissionedPool.sol

79:             .allocateValidatorsAndUpdateOperatorId(requiredValidators);

124:                 nodeRegistryAddress

127:                 withdrawVaultAddress

246:         bytes32 pubkey_root = sha256(abi.encodePacked(_pubkey, bytes16(0)));

278:             _nodeRegistryAddress

282:             withdrawVaultAddress

```

```solidity
File: example/PermissionlessNodeRegistry.sol

485:         uint256 validatorCount = 0;

525:         for (uint256 i = startIndex; i < endIndex; i++) {

554:         uint256 optOutOperatorCount = 0;

675:         return

685:         return validator.status == ValidatorStatus.DEPOSITED;

```

```solidity
File: example/PermissionlessPool.sol

87: 

211:         bytes32 pubkey_root = sha256(abi.encodePacked(_pubkey, bytes16(0)));

235:             _nodeRegistryAddress

239:             withdrawVaultAddress

```

```solidity
File: example/PoolSelector.sol

113:         uint256 poolCount = poolIdArray.length;

```

```solidity
File: example/StaderOracle.sol

395:         // Get submission keys

454: 

```

```solidity
File: example/StaderStakePoolsManager.sol

212:             staderConfig.getPoolSelector()

```

```solidity
File: example/UserWithdrawalManager.sol

120:             uint256 requiredEth = userWithdrawInfo.ethExpected;

156:         if (msg.sender != userRequest.owner) {

```

```solidity
File: example/UtilLib.sol

52:             _validatorId

```

```solidity
File: example/ValidatorWithdrawalVault.sol

132:         bytes[] memory pubkeyArray = new bytes[](1);

133:         pubkeyArray[0] = pubkey;

```

```solidity
File: example/VaultProxy.sol

43:         if (!success) {

```

### [GAS-9] Increments can be `unchecked` in for-loops

*Instances (15)*:
```solidity
File: example/PermissionedNodeRegistry.sol

75:         for (uint256 i = 0; i < permissionedNosLength; i++) {

582:         for (uint256 i = startIndex; i < endIndex; i++) {

621:         for (uint256 i = startIndex; i < endIndex; i++) {

```

```solidity
File: example/PermissionlessNodeRegistry.sol

486:         for (uint256 i = startIndex; i < endIndex; i++) {

525:         for (uint256 i = startIndex; i < endIndex; i++) {

555:         for (uint256 i = startIndex; i < endIndex; i++) {

```

```solidity
File: example/PoolSelector.sol

121:         for (uint256 i = 0; i < poolTargetLength; i++) {

```

```solidity
File: example/PoolUtils.sol

96:         for (uint256 i = 0; i < poolCount; i++) {

160:         for (uint256 i = 0; i < poolCount; i++) {

171:         for (uint256 i = 0; i < poolCount; i++) {

182:         for (uint256 i = 0; i < poolCount; i++) {

193:         for (uint256 i = 0; i < poolCount; i++) {

265:         for (uint256 i = 0; i < poolCount; i++) {

```

```solidity
File: example/SocializingPool.sol

133:         for (uint256 i = 0; i < indexLength; i++) {

```

```solidity
File: example/StaderStakePoolsManager.sol

216:         for (uint256 i = 0; i < poolCount; i++) {

```

### [GAS-10] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (13)*:
```solidity
File: example/Auction.sol

99:         if (lotItem.highestBidAmount > 0) revert LotWasAuctioned();

```

```solidity
File: example/PermissionedNodeRegistry.sol

188:         bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);

283:         if (totalDefectedKeys > 0) {

```

```solidity
File: example/PermissionlessNodeRegistry.sol

191:         if (frontRunValidatorsLength > 0) {

287:             if (address(feeRecipientAddress).balance > 0) {

```

```solidity
File: example/SocializingPool.sol

107:         if (totalAmountETH > 0) {

115:         if (totalAmountSD > 0) {

```

```solidity
File: example/StaderOracle.sol

108:         if (_exchangeRate.reportingBlockNumber % updateFrequencyMap[ETHX_ER_UF] > 0) {

261:         if (_sdPriceData.reportingBlockNumber % updateFrequencyMap[SD_PRICE_UF] > 0) {

315:         if (_validatorStats.reportingBlockNumber % updateFrequencyMap[VALIDATOR_STATS_UF] > 0) {

390:         if (_withdrawnValidators.reportingBlockNumber % updateFrequencyMap[WITHDRAWN_VALIDATORS_UF] > 0) {

```

```solidity
File: example/StaderStakePoolsManager.sol

314:             (totalAssets() > 0 ||

```

```solidity
File: example/ValidatorWithdrawalVault.sol

101:         if (totalRewards > 0) {

```

### [GAS-11] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (14)*:
```solidity
File: example/UtilLib.sol

16:     function checkNonZeroAddress(address _address) internal pure {

21:     function onlyManagerRole(address _addr, IStaderConfig _staderConfig) internal view {

27:     function onlyOperatorRole(address _addr, IStaderConfig _staderConfig) internal view {

34:     function onlyStaderContract(

44:     function getPubkeyForValidSender(

60:     function getOperatorForValidSender(

77:     function onlyValidatorWithdrawVault(

90:     function getOperatorAddressByValidatorId(

102:     function getOperatorAddressByOperatorId(

113:     function getOperatorRewardAddress(address _operator, IStaderConfig _staderConfig)

129:     function getPubkeyRoot(bytes calldata _pubkey) internal pure returns (bytes32) {

138:     function getValidatorSettleStatus(bytes calldata _pubkey, IStaderConfig _staderConfig)

150:     function computeExchangeRate(

162:     function sendValue(address _receiver, uint256 _amount) internal {

```