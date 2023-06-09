| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 2 |
| [GAS-2](#GAS-2) | `abi.encode()` is less efficient than `abi.encodepacked()` | 18 |
| [GAS-3](#GAS-3) | `keccak256()` EXPRESSIONS WHICH ARE FIXED, CAN BE PRECALCULATED AND ASSIGNED TO THE CONSTANT VARIABLES. | 47 |
| [GAS-4](#GAS-4) | Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead | 89 |
| [GAS-5](#GAS-5) | Use nested if and, avoid multiple check combinations | 2 |
| [GAS-6](#GAS-6) | Using `private` rather than `public` for constants, saves gas | 63 |
| [GAS-7](#GAS-7) | Don't initialize variables with default value | 13 |
| [GAS-8](#GAS-8) | 10**18, can be changed to 1e18 and save some gas | 3 |
| [GAS-9](#GAS-9) | Inverting the condition of an if-else-statement wastes gas | 11 |
| [GAS-10](#GAS-10) | Gas saving is achieved by removing the `delete` keyword (~60k) | 2 |
| [GAS-11](#GAS-11) | Use double `if` statements instead of `&&` | 2 |
| [GAS-12](#GAS-12) | With assembly, `.call (bool success)` transfer can be done gas-optimized | 7 |
| [GAS-13](#GAS-13) | Use constants instead of `type(uintx).max` | 1 |
| [GAS-14](#GAS-14) | Use `!= 0` instead of `> 0` for unsigned integer comparison | 13 |
| [GAS-15](#GAS-15) | Use shift Right/Left instead of division/multiplication if possible | 49 |
| [GAS-16](#GAS-16) | Write direct outcome, instead of performing mathematical operations  | 1 |
| [GAS-17](#GAS-17) | Use assembly to hash instead of Solidity | 12 |


### **[GAS-1]** Use assembly to check for `address(0)`

Saves 6 gas per instance if using assembly to check for address(0)

```solidity
File:2023-06-stader/contracts/UserWithdrawalManager.sol
96:        if (_owner == address(0)) revert ZeroAddressReceived();
```
```solidity
File:2023-06-stader/contracts/library/UtilLib.sol
22:        if (_address == address(0)) revert ZeroAddress();
```

*Instances (2)*


### **[GAS-2]** `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/StaderOracle.sol
127:            abi.encode(
135:            abi.encode(_exchangeRate.reportingBlockNumber, _exchangeRate.totalETHBalance, _exchangeRate.totalETHXSupply)
221:            abi.encode(
233:            abi.encode(
282:        bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _sdPriceData.reportingBlockNumber));
283:        bytes32 submissionCountKey = keccak256(abi.encode(_sdPriceData.reportingBlockNumber));
334:            abi.encode(
346:            abi.encode(
407:        bytes memory encodedPubkeys = abi.encode(_withdrawnValidators.sortedPubkeys);
410:            abi.encode(
418:            abi.encode(_withdrawnValidators.reportingBlockNumber, _withdrawnValidators.nodeRegistry, encodedPubkeys)
466:        bytes memory encodedPubkeys = abi.encode(_mapd.sortedPubkeys);
469:        bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _mapd.index, encodedPubkeys));
470:        bytes32 submissionCountKey = keccak256(abi.encode(_mapd.index, encodedPubkeys));
```
```solidity
File:2023-06-stader/contracts/factory/VaultFactory.sol
40:        bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));
54:        bytes32 salt = sha256(abi.encode(_poolId, _operatorId));
67:        bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));
77:        bytes32 salt = sha256(abi.encode(_poolId, _operatorId));
```
</details>
*Instances (18)*



### **[GAS-3]** `keccak256()` EXPRESSIONS WHICH ARE FIXED, CAN BE PRECALCULATED AND ASSIGNED TO THE CONSTANT VARIABLES.

Instead of calculating the keccak256() when the contract is made, pre calculate them before and only give the result to a constant
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/ETHx.sol
21:    bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');
22:    bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');
```
```solidity
File:2023-06-stader/contracts/StaderConfig.sol
12:    bytes32 public constant ETH_PER_NODE = keccak256('ETH_PER_NODE');
14:    bytes32 public constant PRE_DEPOSIT_SIZE = keccak256('PRE_DEPOSIT_SIZE');
16:    bytes32 public constant FULL_DEPOSIT_SIZE = keccak256('FULL_DEPOSIT_SIZE');
18:    bytes32 public constant DECIMALS = keccak256('DECIMALS');
20:    bytes32 public constant TOTAL_FEE = keccak256('TOTAL_FEE');
22:    bytes32 public constant OPERATOR_MAX_NAME_LENGTH = keccak256('OPERATOR_MAX_NAME_LENGTH');
24:    bytes32 public constant SOCIALIZING_POOL_CYCLE_DURATION = keccak256('SOCIALIZING_POOL_CYCLE_DURATION');
27:    bytes32 public constant REWARD_THRESHOLD = keccak256('REWARD_THRESHOLD');
28:    bytes32 public constant MIN_DEPOSIT_AMOUNT = keccak256('MIN_DEPOSIT_AMOUNT');
29:    bytes32 public constant MAX_DEPOSIT_AMOUNT = keccak256('MAX_DEPOSIT_AMOUNT');
30:    bytes32 public constant MIN_WITHDRAW_AMOUNT = keccak256('MIN_WITHDRAW_AMOUNT');
31:    bytes32 public constant MAX_WITHDRAW_AMOUNT = keccak256('MAX_WITHDRAW_AMOUNT');
35:    bytes32 public constant WITHDRAWN_KEYS_BATCH_SIZE = keccak256('WITHDRAWN_KEYS_BATCH_SIZE');
37:    bytes32 public constant ADMIN = keccak256('ADMIN');
38:    bytes32 public constant STADER_TREASURY = keccak256('STADER_TREASURY');
40:    bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');
41:    bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');
42:    bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');
43:    bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');
44:    bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');
45:    bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');
46:    bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');
47:    bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');
48:    bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');
49:    bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');
50:    bytes32 public constant override ETH_DEPOSIT_CONTRACT = keccak256('ETH_DEPOSIT_CONTRACT');
51:    bytes32 public constant override PERMISSIONLESS_POOL = keccak256('PERMISSIONLESS_POOL');
52:    bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');
53:    bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');
54:    bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');
55:    bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');
56:    bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');
57:    bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');
64:    bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');
65:    bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');
68:    bytes32 public constant override MANAGER = keccak256('MANAGER');
69:    bytes32 public constant override OPERATOR = keccak256('OPERATOR');
71:    bytes32 public constant SD = keccak256('SD');
72:    bytes32 public constant ETHx = keccak256('ETHx');
```
```solidity
File:2023-06-stader/contracts/StaderOracle.sol
50:    bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency
51:    bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key
52:    bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key
53:    bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key
54:    bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key
```
```solidity
File:2023-06-stader/contracts/factory/VaultFactory.sol
16:    bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');
```
</details>
*Instances (47)*


### **[GAS-4]** Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/NodeELRewardVault.sol
25:        uint8 poolId = IVaultProxy(address(this)).poolId();
```
```solidity
File:2023-06-stader/contracts/Penalty.sol
144:    function markValidatorSettled(uint8 _poolId, uint256 _validatorId) external override {
```
```solidity
File:2023-06-stader/contracts/PermissionedNodeRegistry.sol
31:    uint8 public constant override POOL_ID = 2;
32:    uint16 public override inputKeyCountLimit;
33:    uint64 public override maxNonTerminalKeyPerOperator;
416:    function updateMaxNonTerminalKeyPerOperator(uint64 _maxNonTerminalKeyPerOperator) external override {
427:    function updateInputKeyCountLimit(uint16 _inputKeyCountLimit) external override {
526:    ) public view override returns (uint64) {
533:        uint64 totalNonWithdrawnKeyCount;
```
```solidity
File:2023-06-stader/contracts/PermissionedPool.sol
322:        uint64 value = uint64(_depositAmount / 1 gwei);
```
```solidity
File:2023-06-stader/contracts/PermissionlessNodeRegistry.sol
30:    uint8 public constant override POOL_ID = 1;
31:    uint16 public override inputKeyCountLimit;
32:    uint64 public override maxNonTerminalKeyPerOperator;
325:    function updateInputKeyCountLimit(uint16 _inputKeyCountLimit) external override {
336:    function updateMaxNonTerminalKeyPerOperator(uint64 _maxNonTerminalKeyPerOperator) external override {
407:    ) public view override returns (uint64) {
414:        uint64 totalNonWithdrawnKeyCount;
```
```solidity
File:2023-06-stader/contracts/PermissionlessPool.sol
274:        uint64 value = uint64(_depositAmount / 1 gwei);
```
```solidity
File:2023-06-stader/contracts/PoolSelector.sol
18:    uint16 public poolAllocationMaxSize;
23:    mapping(uint8 => uint256) public poolWeights;
50:    function computePoolAllocationForDeposit(uint8 _poolId, uint256 _newValidatorToRegister)
79:        returns (uint256[] memory selectedPoolCapacity, uint8[] memory poolIdArray)
121:        uint8[] memory poolIdArray = IPoolUtils(staderConfig.getPoolUtils()).getPoolIdArray();
140:    function updatePoolAllocationMaxSize(uint16 _poolAllocationMaxSize) external {
```
```solidity
File:2023-06-stader/contracts/PoolUtils.sol
13:    uint64 private constant PUBKEY_LENGTH = 48;
14:    uint64 private constant SIGNATURE_LENGTH = 96;
17:    mapping(uint8 => address) public override poolAddressById;
18:    uint8[] public override poolIdArray;
40:    function addNewPool(uint8 _poolId, address _poolAddress) external override {
55:    function updatePoolAddress(uint8 _poolId, address _newPoolAddress)
91:    function getProtocolFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {
96:    function getOperatorFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {
112:    function getQueuedValidatorCountByPool(uint8 _poolId)
124:    function getActiveValidatorCountByPool(uint8 _poolId)
136:    function getSocializingPoolAddress(uint8 _poolId)
148:        uint8 _poolId,
157:    function getCollateralETH(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {
162:    function getNodeRegistry(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (address) {
188:    function getOperatorPoolId(address _operAddr) external view override returns (uint8) {
199:    function getValidatorPoolId(bytes calldata _pubkey) external view override returns (uint8) {
210:    function getPoolIdArray() external view override returns (uint8[] memory) {
245:    function calculateRewardShare(uint8 _poolId, uint256 _totalRewards)
271:    function isExistingPoolId(uint8 _poolId) public view override returns (bool) {
281:    function verifyNewPool(uint8 _poolId, address _poolAddress) internal view {
295:    modifier onlyExistingPoolId(uint8 _poolId) {
```
```solidity
File:2023-06-stader/contracts/SDCollateral.sol
18:    mapping(uint8 => PoolThresholdInfo) public poolThresholdbyPoolId;
78:    function slashValidatorSD(uint256 _validatorId, uint8 _poolId) external override nonReentrant {
120:        uint8 _poolId,
145:        (uint8 poolId, , uint256 validatorCount) = getOperatorInfo(_operator);
157:        uint8 _poolId,
166:    function getMinimumSDToBond(uint8 _poolId, uint256 _numValidator)
185:        uint8 _poolId,
194:        (uint8 poolId, , uint256 validatorCount) = getOperatorInfo(_operator);
221:            uint8 _poolId,
238:    function isPoolThresholdValid(uint8 _poolId) internal view {
```
```solidity
File:2023-06-stader/contracts/StaderOracle.sol
46:    mapping(bytes32 => uint8) private submissionCountKeys;
47:    mapping(bytes32 => uint16) public override missedAttestationPenalty;
137:        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);
253:        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);
284:        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);
357:        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);
421:        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);
471:        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);
575:    function getMerkleRootReportableBlockByPoolId(uint8 _poolId) public view override returns (uint256) {
606:    function getCurrentRewardsIndexByPoolId(uint8 _poolId) public view returns (uint256) {
622:        returns (uint8 _submissionCount)
```
```solidity
File:2023-06-stader/contracts/StaderStakePoolsManager.sol
90:    function receiveExcessEthFromPool(uint8 _poolId) external payable override {
183:    function validatorBatchDeposit(uint8 _poolId) external override nonReentrant whenNotPaused {
227:        (uint256[] memory selectedPoolCapacity, uint8[] memory poolIdArray) = IPoolSelector(
```
```solidity
File:2023-06-stader/contracts/ValidatorWithdrawalVault.sol
31:        uint8 poolId = VaultProxy(payable(address(this))).poolId();
55:        uint8 poolId = VaultProxy(payable(address(this))).poolId();
94:        uint8 poolId = VaultProxy(payable(address(this))).poolId();
127:    function getCollateralETH(uint8 _poolId, IStaderConfig _staderConfig) internal view returns (uint256) {
132:        uint8 _poolId,
140:        uint8 _poolId,
```
```solidity
File:2023-06-stader/contracts/VaultProxy.sol
12:    uint8 public override poolId;
22:        uint8 _poolId,
```
```solidity
File:2023-06-stader/contracts/factory/VaultFactory.sol
35:        uint8 _poolId,
48:    function deployNodeELRewardVault(uint8 _poolId, uint256 _operatorId)
63:        uint8 _poolId,
71:    function computeNodeELRewardVaultAddress(uint8 _poolId, uint256 _operatorId)
```
```solidity
File:2023-06-stader/contracts/library/UtilLib.sol
18:    uint64 private constant VALIDATOR_PUBKEY_LENGTH = 48;
50:        uint8 _poolId,
66:        uint8 _poolId,
83:        uint8 _poolId,
96:        uint8 _poolId,
108:        uint8 _poolId,
123:        uint8 poolId = IPoolUtils(_staderConfig.getPoolUtils()).getOperatorPoolId(_operator);
148:        uint8 poolId = IPoolUtils(_staderConfig.getPoolUtils()).getValidatorPoolId(_pubkey);
```
</details>
*Instances (89)*

### **[GAS-5]** Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity
File:2023-06-stader/contracts/SocializingPool.sol
146:            if (_amountSD[i] == 0 && _amountETH[i] == 0) {
```
```solidity
File:2023-06-stader/contracts/ValidatorWithdrawalVault.sol
35:        if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
```

*Instances (2)*

### **[GAS-6]** Using `private` rather than `public` for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/Auction.sol
22:    uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours
```
```solidity
File:2023-06-stader/contracts/ETHx.sol
21:    bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');
22:    bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');
```
```solidity
File:2023-06-stader/contracts/PermissionedNodeRegistry.sol
31:    uint8 public constant override POOL_ID = 2;
```
```solidity
File:2023-06-stader/contracts/PermissionedPool.sol
33:    uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;
```
```solidity
File:2023-06-stader/contracts/PermissionlessNodeRegistry.sol
30:    uint8 public constant override POOL_ID = 1;
42:    uint256 public constant override FRONT_RUN_PENALTY = 3 ether;
43:    uint256 public constant COLLATERAL_ETH = 4 ether;
```
```solidity
File:2023-06-stader/contracts/PermissionlessPool.sol
23:    uint256 public constant DEPOSIT_NODE_BOND = 3 ether;
31:    uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;
```
```solidity
File:2023-06-stader/contracts/PoolSelector.sol
21:    uint256 public constant POOL_WEIGHTS_SUM = 10000;
```
```solidity
File:2023-06-stader/contracts/StaderConfig.sol
12:    bytes32 public constant ETH_PER_NODE = keccak256('ETH_PER_NODE');
14:    bytes32 public constant PRE_DEPOSIT_SIZE = keccak256('PRE_DEPOSIT_SIZE');
16:    bytes32 public constant FULL_DEPOSIT_SIZE = keccak256('FULL_DEPOSIT_SIZE');
18:    bytes32 public constant DECIMALS = keccak256('DECIMALS');
20:    bytes32 public constant TOTAL_FEE = keccak256('TOTAL_FEE');
22:    bytes32 public constant OPERATOR_MAX_NAME_LENGTH = keccak256('OPERATOR_MAX_NAME_LENGTH');
24:    bytes32 public constant SOCIALIZING_POOL_CYCLE_DURATION = keccak256('SOCIALIZING_POOL_CYCLE_DURATION');
25:    bytes32 public constant SOCIALIZING_POOL_OPT_IN_COOLING_PERIOD =
27:    bytes32 public constant REWARD_THRESHOLD = keccak256('REWARD_THRESHOLD');
28:    bytes32 public constant MIN_DEPOSIT_AMOUNT = keccak256('MIN_DEPOSIT_AMOUNT');
29:    bytes32 public constant MAX_DEPOSIT_AMOUNT = keccak256('MAX_DEPOSIT_AMOUNT');
30:    bytes32 public constant MIN_WITHDRAW_AMOUNT = keccak256('MIN_WITHDRAW_AMOUNT');
31:    bytes32 public constant MAX_WITHDRAW_AMOUNT = keccak256('MAX_WITHDRAW_AMOUNT');
33:    bytes32 public constant MIN_BLOCK_DELAY_TO_FINALIZE_WITHDRAW_REQUEST =
35:    bytes32 public constant WITHDRAWN_KEYS_BATCH_SIZE = keccak256('WITHDRAWN_KEYS_BATCH_SIZE');
37:    bytes32 public constant ADMIN = keccak256('ADMIN');
38:    bytes32 public constant STADER_TREASURY = keccak256('STADER_TREASURY');
40:    bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');
41:    bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');
42:    bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');
43:    bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');
44:    bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');
45:    bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');
46:    bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');
47:    bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');
48:    bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');
49:    bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');
50:    bytes32 public constant override ETH_DEPOSIT_CONTRACT = keccak256('ETH_DEPOSIT_CONTRACT');
51:    bytes32 public constant override PERMISSIONLESS_POOL = keccak256('PERMISSIONLESS_POOL');
52:    bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');
53:    bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');
54:    bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');
55:    bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');
56:    bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');
57:    bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');
58:    bytes32 public constant override NODE_EL_REWARD_VAULT_IMPLEMENTATION =
60:    bytes32 public constant override VALIDATOR_WITHDRAWAL_VAULT_IMPLEMENTATION =
64:    bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');
65:    bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');
68:    bytes32 public constant override MANAGER = keccak256('MANAGER');
69:    bytes32 public constant override OPERATOR = keccak256('OPERATOR');
71:    bytes32 public constant SD = keccak256('SD');
72:    bytes32 public constant ETHx = keccak256('ETHx');
```
```solidity
File:2023-06-stader/contracts/StaderOracle.sol
26:    uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days
27:    uint256 public constant ER_CHANGE_MAX_BPS = 10000;
29:    uint256 public constant MIN_TRUSTED_NODES = 5;
50:    bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency
51:    bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key
52:    bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key
53:    bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key
54:    bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key
```
```solidity
File:2023-06-stader/contracts/factory/VaultFactory.sol
16:    bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');
```
</details>
*Instances (63)*

### **[GAS-7]** Don't initialize variables with default value

If a variable is not initialized, it is assumed to have the default value. Explicitly initializing a variable with its default value costs unnecessary gas. For more info, see [Mudit Gupta's Blog](https://mudit.blog/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size/) at point `No need to initialize variables with default values` .
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/PermissionedNodeRegistry.sol
91:        for (uint256 i = 0; i < permissionedNosLength; i++) {
597:        uint256 validatorCount = 0;
```
```solidity
File:2023-06-stader/contracts/PermissionlessNodeRegistry.sol
503:        uint256 validatorCount = 0;
572:        uint256 optOutOperatorCount = 0;
```
```solidity
File:2023-06-stader/contracts/PoolSelector.sol
130:        for (uint256 i = 0; i < poolTargetLength; i++) {
```
```solidity
File:2023-06-stader/contracts/PoolUtils.sol
104:        for (uint256 i = 0; i < poolCount; i++) {
168:        for (uint256 i = 0; i < poolCount; i++) {
179:        for (uint256 i = 0; i < poolCount; i++) {
190:        for (uint256 i = 0; i < poolCount; i++) {
201:        for (uint256 i = 0; i < poolCount; i++) {
273:        for (uint256 i = 0; i < poolCount; i++) {
```
```solidity
File:2023-06-stader/contracts/SocializingPool.sol
145:        for (uint256 i = 0; i < indexLength; i++) {
```
```solidity
File:2023-06-stader/contracts/StaderStakePoolsManager.sol
232:        for (uint256 i = 0; i < poolCount; i++) {
```
</details>
*Instances (13)*

### **[GAS-8]** 10**18, can be changed to 1e18 and save some gas

10**18 can be changed to 1e18 to avoid unnecessary arithmetic operation and save gas.

```solidity
File:2023-06-stader/contracts/StaderConfig.sol
93:        setConstant(DECIMALS, 10**18);
95:        setVariable(MIN_DEPOSIT_AMOUNT, 10**14);
97:        setVariable(MIN_WITHDRAW_AMOUNT, 10**14);
```

*Instances (3)*

### **[GAS-9]** Inverting the condition of an if-else-statement wastes gas

Flipping the `true` and `false` blocks instead saves 3 gas
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/PermissionedNodeRegistry.sol
532:        _endIndex = _endIndex > validatorCount ? validatorCount : _endIndex;
595:        endIndex = endIndex > nextValidatorId ? nextValidatorId : endIndex;
635:        endIndex = endIndex > validatorCount ? validatorCount : endIndex;
636:        Validator[] memory validators = new Validator[](endIndex > startIndex ? endIndex - startIndex : 0);
```
```solidity
File:2023-06-stader/contracts/PermissionlessNodeRegistry.sol
413:        _endIndex = _endIndex > validatorCount ? validatorCount : _endIndex;
501:        endIndex = endIndex > nextValidatorId ? nextValidatorId : endIndex;
541:        endIndex = endIndex > validatorCount ? validatorCount : endIndex;
542:        Validator[] memory validators = new Validator[](endIndex > startIndex ? endIndex - startIndex : 0);
570:        endIndex = endIndex > nextOperatorId ? nextOperatorId : endIndex;
```
```solidity
File:2023-06-stader/contracts/SDCollateral.sol
190:        return (sdBalance >= minSDToBond ? 0 : minSDToBond - sdBalance);
```
```solidity
File:2023-06-stader/contracts/StaderStakePoolsManager.sol
297:            (supply == 0) ? initialConvertToAssets(_shares, rounding) : _shares.mulDiv(totalAssets(), supply, rounding);
```
</details>
*Instances (11)*


### **[GAS-10]** Gas saving is achieved by removing the `delete` keyword (~60k)

30k gas savings were made by removing the `delete` keyword. The reason for using the `delete` keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the `delete` key word is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity
File:2023-06-stader/contracts/StaderOracle.sol
293:            delete sdPrices;
```
```solidity
File:2023-06-stader/contracts/UserWithdrawalManager.sol
207:        delete (userWithdrawRequests[_requestId]);
```

*Instances (2)*

### **[GAS-11]** Use double `if` statements instead of `&&`

If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/SocializingPool.sol
146:            if (_amountSD[i] == 0 && _amountETH[i] == 0) {
```
```solidity
File:2023-06-stader/contracts/ValidatorWithdrawalVault.sol
35:        if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
```
</details>
*Instances (2)*

### **[GAS-12]** With assembly, `.call (bool success)` transfer can be done gas-optimized

`return` data `(bool success,)` has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/Auction.sol
131:        (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');
```
```solidity
File:2023-06-stader/contracts/SocializingPool.sol
91:        (bool success, ) = payable(staderConfig.getStaderTreasury()).call{value: _rewardsData.protocolETHRewards}('');
121:            (success, ) = payable(operatorRewardsAddr).call{value: totalAmountETH}('');
```
```solidity
File:2023-06-stader/contracts/StaderInsuranceFund.sol
48:        (bool success, ) = payable(msg.sender).call{value: _amount}('');
```
```solidity
File:2023-06-stader/contracts/StaderStakePoolsManager.sol
102:        (bool success, ) = payable(staderConfig.getUserWithdrawManager()).call{value: _amount}('');
```
```solidity
File:2023-06-stader/contracts/UserWithdrawalManager.sol
229:        (bool success, ) = _recipient.call{value: _amount}('');
```
```solidity
File:2023-06-stader/contracts/library/UtilLib.sol
168:        (bool success, ) = payable(_receiver).call{value: _amount}('');
```
</details>
*Instances (7)*

### **[GAS-13]** Use constants instead of `type(uintx).max`

it uses more gas in the distribution process and also for each transaction than constant usage.
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/SDCollateral.sol
106:        IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);
```
</details>
*Instances (1)*

### **[GAS-14]** Use `!= 0` instead of `> 0` for unsigned integer comparison

It is cheaper to use `!= 0` than > `0` for uint256.
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/Auction.sol
109:        if (lotItem.highestBidAmount > 0) revert LotWasAuctioned();
```
```solidity
File:2023-06-stader/contracts/PermissionedNodeRegistry.sol
204:        bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);
299:        if (totalDefectedKeys > 0) {
```
```solidity
File:2023-06-stader/contracts/PermissionlessNodeRegistry.sol
209:        if (frontRunValidatorsLength > 0) {
305:            if (address(feeRecipientAddress).balance > 0) {
```
```solidity
File:2023-06-stader/contracts/SocializingPool.sol
119:        if (totalAmountETH > 0) {
127:        if (totalAmountSD > 0) {
```
```solidity
File:2023-06-stader/contracts/StaderOracle.sol
121:        if (_exchangeRate.reportingBlockNumber % updateFrequencyMap[ETHX_ER_UF] > 0) {
274:        if (_sdPriceData.reportingBlockNumber % updateFrequencyMap[SD_PRICE_UF] > 0) {
328:        if (_validatorStats.reportingBlockNumber % updateFrequencyMap[VALIDATOR_STATS_UF] > 0) {
403:        if (_withdrawnValidators.reportingBlockNumber % updateFrequencyMap[WITHDRAWN_VALIDATORS_UF] > 0) {
```
```solidity
File:2023-06-stader/contracts/StaderStakePoolsManager.sol
330:            (totalAssets() > 0 ||
```
```solidity
File:2023-06-stader/contracts/ValidatorWithdrawalVault.sol
115:        if (totalRewards > 0) {
```
</details>
*Instances (13)*

### **[GAS-15]** Use shift Right/Left instead of division/multiplication if possible

A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left. While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/Auction.sol
38:        duration = 2 * MIN_AUCTION_DURATION;
```
```solidity
File:2023-06-stader/contracts/Penalty.sol
128:        return violatedEpochs.length * mevTheftPenaltyPerStrike;
```
```solidity
File:2023-06-stader/contracts/PermissionedNodeRegistry.sol
201:        uint256 validatorPerOperator = _numValidators / totalActiveOperatorCount;
593:        uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;
628:        uint256 startIndex = (_pageNumber - 1) * _pageSize;
```
```solidity
File:2023-06-stader/contracts/PermissionedPool.sol
73:            _defectiveKeyCount * staderConfig.getPreDepositSize()
77:        uint256 amountToSendToPoolManager = _defectiveKeyCount * staderConfig.getStakedEthPerNode();
91:        uint256 requiredValidators = msg.value / staderConfig.getStakedEthPerNode();
322:        uint64 value = uint64(_depositAmount / 1 gwei);
```
```solidity
File:2023-06-stader/contracts/PermissionlessNodeRegistry.sol
170:            value: staderConfig.getPreDepositSize() * keyCount
211:                value: frontRunValidatorsLength * FRONT_RUN_PENALTY
499:        uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;
534:        uint256 startIndex = (_pageNumber - 1) * _pageSize;
568:        uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;
664:        if (msg.value != keyCount * COLLATERAL_ETH) {
```
```solidity
File:2023-06-stader/contracts/PermissionlessPool.sol
128:        uint256 requiredValidators = msg.value / (staderConfig.getFullDepositSize() - DEPOSIT_NODE_BOND);
131:            requiredValidators * DEPOSIT_NODE_BOND
274:        uint64 value = uint64(_depositAmount / 1 gwei);
```
```solidity
File:2023-06-stader/contracts/PoolSelector.sol
61:        uint256 poolTotalTarget = (poolWeights[_poolId] * totalValidatorsRequired) / POOL_WEIGHTS_SUM;
93:            uint256 remainingValidatorsToDeposit = ethToDeposit / poolDepositSize;
99:            ethToDeposit -= selectedPoolCapacity[i] * poolDepositSize;
```
```solidity
File:2023-06-stader/contracts/PoolUtils.sol
261:        uint256 _userShareBeforeCommision = (_totalRewards * usersETH) / TOTAL_STAKED_ETH;
263:        protocolShare = (protocolFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();
265:        operatorShare = (_totalRewards * collateralETH) / TOTAL_STAKED_ETH;
266:        operatorShare += (operatorFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();
```
```solidity
File:2023-06-stader/contracts/SDCollateral.sol
148:        return convertETHToSD(poolThreshold.withdrawThreshold * validatorCount);
199:        uint256 totalMinThreshold = validatorCount * convertETHToSD(poolThreshold.minThreshold);
200:        uint256 totalMaxThreshold = validatorCount * convertETHToSD(poolThreshold.maxThreshold);
207:        return (_sdAmount * sdPriceInETH) / staderConfig.getDecimals();
212:        return (_ethAmount * staderConfig.getDecimals()) / sdPriceInETH;
```
```solidity
File:2023-06-stader/contracts/StaderOracle.sol
26:    uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days
148:            submissionCount == trustedNodesCount / 2 + 1 &&
255:        if ((submissionCount == trustedNodesCount / 2 + 1)) {
290:        if ((submissionCount == (2 * trustedNodesCount) / 3 + 1)) {
314:        return (dataArray[(len - 1) / 2] + dataArray[len / 2]) / 2;
372:            submissionCount == trustedNodesCount / 2 + 1 &&
432:            submissionCount == trustedNodesCount / 2 + 1 &&
482:        if ((submissionCount == trustedNodesCount / 2 + 1)) {
603:        return (block.number / updateFrequency) * updateFrequency;
665:            !(newExchangeRate >= (currentExchangeRate * (ER_CHANGE_MAX_BPS - erChangeLimit)) / ER_CHANGE_MAX_BPS &&
666:                newExchangeRate <= ((currentExchangeRate * (ER_CHANGE_MAX_BPS + erChangeLimit)) / ER_CHANGE_MAX_BPS))
```
```solidity
File:2023-06-stader/contracts/StaderStakePoolsManager.sol
57:        excessETHDepositCoolDown = 3 * 7200;
199:            (availableETHForNewDeposit / poolDepositSize)
207:        IStaderPoolBase(poolAddress).stakeUserETHToBeaconChain{value: selectedPoolCapacity * poolDepositSize}();
208:        emit ETHTransferredToPool(_poolId, poolAddress, selectedPoolCapacity * poolDepositSize);
243:            IStaderPoolBase(poolAddress).stakeUserETHToBeaconChain{value: validatorToDeposit * poolDepositSize}();
244:            emit ETHTransferredToPool(i, poolAddress, validatorToDeposit * poolDepositSize);
```
```solidity
File:2023-06-stader/contracts/UserWithdrawalManager.sol
136:            uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);
```
```solidity
File:2023-06-stader/contracts/library/UtilLib.sol
163:            : (totalETHBalance * DECIMALS) / totalETHXSupply;
```
</details>

*Instances (49)*


### **[GAS-16]** Write direct outcome, instead of performing mathematical operations 

Write direct outcome instead ofperforming mathematical operations saves some gas

```solidity
File:2023-06-stader/contracts/StaderOracle.sol
26:    uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days
```

*Instances (1)*

### **[GAS-17]** Use assembly to hash instead of Solidity

hash can be done by use of assembly instead of keccak256(abi.encodePacked()) [see](https://github.com/code-423n4/2023-01-astaria-findings/blob/8059d5e7c4b3b7155259c7990226c91dc2f375fb/data/kaden-G.md#use-assembly-to-hash-instead-of-solidity)

```solidity
File:2023-06-stader/contracts/PermissionedPool.sol
263:        bytes32 pubkey_root = sha256(abi.encodePacked(_pubkey, bytes16(0)));
266:                sha256(abi.encodePacked(_signature[:64])),
267:                sha256(abi.encodePacked(_signature[64:], bytes32(0)))
273:                    sha256(abi.encodePacked(pubkey_root, _withdrawCredential)),
274:                    sha256(abi.encodePacked(amount, bytes24(0), signature_root))
```
```solidity
File:2023-06-stader/contracts/PermissionlessPool.sol
225:        bytes32 pubkey_root = sha256(abi.encodePacked(_pubkey, bytes16(0)));
228:                sha256(abi.encodePacked(_signature[:64])),
229:                sha256(abi.encodePacked(_signature[64:], bytes32(0)))
235:                    sha256(abi.encodePacked(pubkey_root, _withdrawCredential)),
236:                    sha256(abi.encodePacked(amount, bytes24(0), signature_root))
```
```solidity
File:2023-06-stader/contracts/SocializingPool.sol
174:        bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));
```
```solidity
File:2023-06-stader/contracts/library/UtilLib.sol
140:        return sha256(abi.encodePacked(_pubkey, bytes16(0)));
```

*Instances (12)*
