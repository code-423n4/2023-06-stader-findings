## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol

```solidity
// place this modifer before constructor
75:    modifier onlyOwner() {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

```solidity
// modifiers should come before constructor
697:    modifier checkERInspectionMode() {
704:    modifier trustedNodeOnly() {
711:    modifier checkMinTrustedNodes() {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol

```solidity
// modifier should come after state variables
295:    modifier onlyExistingPoolId(uint8 _poolId) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol

```solidity
// events should come after state variables
18:    event UpdatedStaderConfig(address indexed _staderConfig);
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol

```solidity
// public function coming after external ones
85:    function calculateValidatorWithdrawalShare()
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol

```solidity
// public functions coming after some external and internal ones
125:    function getExchangeRate() public view override returns (uint256) {
135:    function totalAssets() public view override returns (uint256) {
140:    function convertToShares(uint256 _assets) public view override returns (uint256) {
145:    function convertToAssets(uint256 _shares) public view override returns (uint256) {
150:    function maxDeposit() public view override returns (uint256) {
154:    function minDeposit() public view override returns (uint256) {
159:    function previewDeposit(uint256 _assets) public view override returns (uint256) {
164:    function previewWithdraw(uint256 _shares) public view override returns (uint256) {
169:    function deposit(address _receiver) public payable override whenNotPaused returns (uint256) {
328:    function isVaultHealthy() public view override returns (bool) {

// internal functions coming before public and external ones
271:    function _convertToShares(uint256 _assets, Math.Rounding rounding) internal view returns (uint256) {
284:    function initialConvertToShares(
294:    function _convertToAssets(uint256 _shares, Math.Rounding rounding) internal view returns (uint256) {
305:    function initialConvertToAssets(
315:    function _deposit(
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

```solidity
// public functions coming after external and some internal ones
185:    function disableERInspectionMode() public override whenNotPaused {
571:    function getERReportableBlock() public view override returns (uint256) {
575:    function getMerkleRootReportableBlockByPoolId(uint8 _poolId) public view override returns (uint256) {
582:    function getSDPriceReportableBlock() public view override returns (uint256) {
586:    function getValidatorStatsReportableBlock() public view override returns (uint256) {
590:    function getWithdrawnValidatorReportableBlock() public view override returns (uint256) {
594:    function getMissedAttestationPenaltyReportableBlock() public view override returns (uint256) {
606:    function getCurrentRewardsIndexByPoolId(uint8 _poolId) public view returns (uint256) {

// internal functions coming before public and external ones
300:    function insertSDPrice(uint256 _sdPrice) internal {
312:    function getMedianValue(uint256[] storage dataArray) internal view returns (uint256 _medianValue) {
559:    function setUpdateFrequency(bytes32 _key, uint256 _updateFrequency) internal {
598:    function getReportableBlockFor(bytes32 _key) internal view returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol

```solidity
// internal functions coming before public and external ones
476:    function setVariable(bytes32 key, uint256 val) internal {
481:    function setAccount(bytes32 key, address val) internal {
487:    function setContract(bytes32 key, address val) internal {
493:    function setToken(bytes32 key, address val) internal {

// public functions coming after internal and external functions  
504:    function onlyManagerRole(address account) public view override returns (bool) {
508:    function onlyOperatorRole(address account) public view override returns (bool) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol

```solidity
// public functions coming after external ones
163:    function verifyProof(
187:    function getCurrentRewardsIndex() public view returns (uint256 index) {
206:    function getRewardCycleDetails(uint256 _index) public view returns (uint256 _startBlock, uint256 _endBlock) {

// internal function coming before public and external ones
137:    function _claim(
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol

```solidity
// internal function coming before public and external ones
137:    function _claim(

// public function coming after external ones
163:    function verifyProof(
187:    function getCurrentRewardsIndex() public view returns (uint256 index) {
206:    function getRewardCycleDetails(uint256 _index) public view returns (uint256 _startBlock, uint256 _endBlock) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol

```solidity
// public functions should come before external functions.
144:    function getOperatorWithdrawThreshold(address _operator) public view returns (uint256 operatorWithdrawThreshold) {
166:    function getMinimumSDToBond(uint8 _poolId, uint256 _numValidator)
183:    function getRemainingSDToBond(
205:    function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {
210:    function convertETHToSD(uint256 _ethAmount) public view override returns (uint256) {

// internal function before some external ones
90:    function slashSD(address _operator, uint256 _sdToSlash) internal {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol

```solidity
// public functions should come before all the other ones
91:    function getProtocolFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {
96:    function getOperatorFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {
124:    function getActiveValidatorCountByPool(uint8 _poolId)
136:    function getSocializingPoolAddress(uint8 _poolId)
147:    function getOperatorTotalNonTerminalKeys(
157:    function getCollateralETH(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {
162:    function getNodeRegistry(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (address) {
166:    function isExistingPubkey(bytes calldata _pubkey) public view override returns (bool) {
271:    function isExistingPoolId(uint8 _poolId) public view override returns (bool) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol

```solidity
// external view coming before non-view externals
50:    function computePoolAllocationForDeposit(uint8 _poolId, uint256 _newValidatorToRegister)
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol

```solidity
// external non-view function coming after view external ones
210:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

```solidity
// external view coming before non-view externals
316:    function getSocializingPoolStateChangeBlock(uint256 _operatorId) external view returns (uint256) {
448:    function getTotalActiveValidatorCount() external view override returns (uint256) {
452:    function getCollateralETH() external pure override returns (uint256) {
460:    function getOperatorRewardAddress(uint256 _operatorId) external view override returns (address payable) {

// public functions coming after external ones
403:    function getOperatorTotalNonTerminalKeys(
432:    function getOperatorTotalKeys(uint256 _operatorId) public view override returns (uint256 _totalKeys) {
440:    function getTotalQueuedValidatorCount() public view override returns (uint256) {

// internal view functions coming before non-view internals
643:    function checkInputKeysCountAndCollateral(
680:    function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {
691:    function isNonTerminalValidator(uint256 _validatorId) internal view returns (bool) {
701:    function isActiveValidator(uint256 _validatorId) internal view returns (bool) {
712:    function onlyInitializedValidator(uint256 _validatorId) internal view {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol

```solidity
// external non view functions coming after external view ones.
236:    function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {
248:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol

```solidity
// public functions coming after external ones
123:    function calculateMEVTheftPenalty(bytes32 _pubkeyRoot) public override returns (uint256) {
132:    function calculateMissedAttestationPenalty(bytes32 _pubkeyRoot) public view override returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol

```solidity
// internal function coming before external one
77:    function _beforeTokenTransfer(
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/ValidatorStatus.sol

```solidity
4:  enum ValidatorStatus {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol

```solidity
9:    library UtilLib {
21:    function checkNonZeroAddress(address _address) internal pure {
26:    function onlyManagerRole(address _addr, IStaderConfig _staderConfig) internal view {
32:    function onlyOperatorRole(address _addr, IStaderConfig _staderConfig) internal view {
39:    function onlyStaderContract(
49:    function getPubkeyForValidSender(
65:    function getOperatorForValidSender(
82:    function onlyValidatorWithdrawVault(
95:    function getOperatorAddressByValidatorId(
107:    function getOperatorAddressByOperatorId(
118:    function getOperatorRewardAddress(address _operator, IStaderConfig _staderConfig)
143:    function getValidatorSettleStatus(bytes calldata _pubkey, IStaderConfig _staderConfig)
155:    function computeExchangeRate(
165:    function sendValue(address _receiver, uint256 _amount) internal {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol

```solidity
23:    function initialize(address _admin, address _staderConfig) external initializer {
34:    function deployWithdrawVault(
48:    function deployNodeELRewardVault(uint8 _poolId, uint256 _operatorId)
62:    function computeWithdrawVaultAddress(
71:    function computeNodeELRewardVaultAddress(uint8 _poolId, uint256 _operatorId)
81:    function getValidatorWithdrawCredential(address _withdrawVault) external pure override returns (bytes memory) {
86:    function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {
93:    function updateVaultProxyAddress(address _vaultProxyImpl) external override onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol

```solidity
20:    function initialise(
41:    fallback(bytes calldata _input) external payable returns (bytes memory) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol

```solidity
18:   contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
30:    function distributeRewards() external override {
54:    function settleFunds() external override {
85:    function calculateValidatorWithdrawalShare()
127:    function getCollateralETH(uint8 _poolId, IStaderConfig _staderConfig) internal view returns (uint256) {
131:    function getOperatorAddress(
139:    function getUpdatedPenaltyAmount(
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol

```solidity
18: contract UserWithdrawalManager is
54:    function initialize(address _admin, address _staderConfig) public initializer {
84:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
184:    function getRequestIdsByUser(address _owner) external view override returns (uint256[] memory) {
206:    function deleteRequestId(uint256 _requestId, address _owner) internal {
223:    function sendValue(address payable _recipient, uint256 _amount) internal {

// @return missing
95:    function requestWithdraw(uint256 _ethXAmount, address _owner) external override whenNotPaused returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol

```solidity
82:    function receiveEthFromAuction() external payable override {
109:    function updateExcessETHDepositCoolDown(uint256 _excessETHDepositCoolDown) external {
116:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

// @return missing
125:    function getExchangeRate() public view override returns (uint256) {
135:    function totalAssets() public view override returns (uint256) {
328:    function isVaultHealthy() public view override returns (bool) {

140:    function convertToShares(uint256 _assets) public view override returns (uint256) {
145:    function convertToAssets(uint256 _shares) public view override returns (uint256) {
150:    function maxDeposit() public view override returns (uint256) {
154:    function minDeposit() public view override returns (uint256) {
159:    function previewDeposit(uint256 _assets) public view override returns (uint256) {
164:    function previewWithdraw(uint256 _shares) public view override returns (uint256) {
169:    function deposit(address _receiver) public payable override whenNotPaused returns (uint256) {

// @param missing
183:    function validatorBatchDeposit(uint8 _poolId) external override nonReentrant whenNotPaused {
315:    function _deposit(

// @param and @return missing
271:    function _convertToShares(uint256 _assets, Math.Rounding rounding) internal view returns (uint256) {
284:    function initialConvertToShares(
294:    function _convertToAssets(uint256 _shares, Math.Rounding rounding) internal view returns (uint256) {
305:    function initialConvertToAssets(
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

```solidity
17:   contract StaderOracle is IStaderOracle, AccessControlUpgradeable, PausableUpgradeable, ReentrancyGuardUpgradeable {
62:    function initialize(address _admin, address _staderConfig) public initializer {
270:    function submitSDPrice(SDPriceData calldata _sdPriceData) external override trustedNodeOnly checkMinTrustedNodes {
300:    function insertSDPrice(uint256 _sdPrice) internal {
312:    function getMedianValue(uint256[] storage dataArray) internal view returns (uint256 _medianValue) {
503:    function disableSafeMode() external override onlyRole(DEFAULT_ADMIN_ROLE) {
509:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
515:    function setERUpdateFrequency(uint256 _updateFrequency) external override {
523:    function togglePORFeedBasedERData() external override checkERInspectionMode {
530:    function updateERChangeLimit(uint256 _erChangeLimit) external override {
539:    function setSDPriceUpdateFrequency(uint256 _updateFrequency) external override {
544:    function setValidatorStatsUpdateFrequency(uint256 _updateFrequency) external override {
549:    function setWithdrawnValidatorsUpdateFrequency(uint256 _updateFrequency) external override {
554:    function setMissedAttestationPenaltyUpdateFrequency(uint256 _updateFrequency) external override {
559:    function setUpdateFrequency(bytes32 _key, uint256 _updateFrequency) internal {
571:    function getERReportableBlock() public view override returns (uint256) {
575:    function getMerkleRootReportableBlockByPoolId(uint8 _poolId) public view override returns (uint256) {
582:    function getSDPriceReportableBlock() public view override returns (uint256) {
586:    function getValidatorStatsReportableBlock() public view override returns (uint256) {
590:    function getWithdrawnValidatorReportableBlock() public view override returns (uint256) {
594:    function getMissedAttestationPenaltyReportableBlock() public view override returns (uint256) {
598:    function getReportableBlockFor(bytes32 _key) internal view returns (uint256) {
606:    function getCurrentRewardsIndexByPoolId(uint8 _poolId) public view returns (uint256) {
612:    function getValidatorStats() external view override returns (ValidatorStats memory) {
616:    function getExchangeRate() external view override returns (ExchangeRate memory) {
620:    function attestSubmission(bytes32 _nodeSubmissionKey, bytes32 _submissionCountKey)
633:    function getSDPriceInETH() external view override returns (uint256) {
637:    function getPORFeedData()
653:    function updateWithInLimitER(
679:    function _updateExchangeRate(
697:    modifier checkERInspectionMode() {
704:    modifier trustedNodeOnly() {
711:    modifier checkMinTrustedNodes() {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol

```solidity
13: contract StaderInsuranceFund is
26:    function initialize(address _admin, address _staderConfig) public initializer {
41:    function withdrawFund(uint256 _amount) external override nonReentrant {
69:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol

```solidity
10:   contract StaderConfig is IStaderConfig, Initializable, AccessControlUpgradeable {
85:    function initialize(address _admin, address _ethDepositContract) external initializer {
107:    function updateSocializingPoolCycleDuration(uint256 _socializingPoolCycleDuration) external onlyRole(MANAGER) {
111:    function updateSocializingPoolOptInCoolingPeriod(uint256 _SocializePoolOptInCoolingPeriod)
118:    function updateRewardsThreshold(uint256 _rewardsThreshold) external onlyRole(MANAGER) {
158:    function updateMinBlockDelayToFinalizeWithdrawRequest(uint256 _minBlockDelay)
176:    function updateAdmin(address _admin) external onlyRole(DEFAULT_ADMIN_ROLE) {
185:    function updateStaderTreasury(address _staderTreasury) external onlyRole(MANAGER) {
191:    function updatePoolUtils(address _poolUtils) external onlyRole(DEFAULT_ADMIN_ROLE) {
195:    function updatePoolSelector(address _poolSelector) external onlyRole(DEFAULT_ADMIN_ROLE) {
199:    function updateSDCollateral(address _sdCollateral) external onlyRole(DEFAULT_ADMIN_ROLE) {
203:    function updateOperatorRewardsCollector(address _operatorRewardsCollector) external onlyRole(DEFAULT_ADMIN_ROLE) {
207:    function updateVaultFactory(address _vaultFactory) external onlyRole(DEFAULT_ADMIN_ROLE) {
211:    function updateAuctionContract(address _auctionContract) external onlyRole(DEFAULT_ADMIN_ROLE) {
215:    function updateStaderOracle(address _staderOracle) external onlyRole(DEFAULT_ADMIN_ROLE) {
219:    function updatePenaltyContract(address _penaltyContract) external onlyRole(DEFAULT_ADMIN_ROLE) {
223:    function updatePermissionedPool(address _permissionedPool) external onlyRole(DEFAULT_ADMIN_ROLE) {
227:    function updateStakePoolManager(address _stakePoolManager) external onlyRole(DEFAULT_ADMIN_ROLE) {
231:    function updatePermissionlessPool(address _permissionlessPool) external onlyRole(DEFAULT_ADMIN_ROLE) {
235:    function updateUserWithdrawManager(address _userWithdrawManager) external onlyRole(DEFAULT_ADMIN_ROLE) {
239:    function updateStaderInsuranceFund(address _staderInsuranceFund) external onlyRole(DEFAULT_ADMIN_ROLE) {
243:    function updatePermissionedNodeRegistry(address _permissionedNodeRegistry) external onlyRole(DEFAULT_ADMIN_ROLE) {
247:    function updatePermissionlessNodeRegistry(address _permissionlessNodeRegistry)
254:    function updatePermissionedSocializingPool(address _permissionedSocializePool)
261:    function updatePermissionlessSocializingPool(address _permissionlessSocializePool)
268:    function updateNodeELRewardImplementation(address _nodeELRewardVaultImpl) external onlyRole(DEFAULT_ADMIN_ROLE) {
272:    function updateValidatorWithdrawalVaultImplementation(address _validatorWithdrawalVaultImpl)
279:    function updateETHBalancePORFeedProxy(address _ethBalanceProxy) external onlyRole(DEFAULT_ADMIN_ROLE) {
283:    function updateETHXSupplyPORFeedProxy(address _ethXSupplyProxy) external onlyRole(DEFAULT_ADMIN_ROLE) {
287:    function updateStaderToken(address _staderToken) external onlyRole(DEFAULT_ADMIN_ROLE) {
291:    function updateETHxToken(address _ethX) external onlyRole(DEFAULT_ADMIN_ROLE) {
297:    function getStakedEthPerNode() external view override returns (uint256) {
301:    function getPreDepositSize() external view override returns (uint256) {
305:    function getFullDepositSize() external view override returns (uint256) {
309:    function getDecimals() external view override returns (uint256) {
313:    function getTotalFee() external view override returns (uint256) {
317:    function getOperatorMaxNameLength() external view override returns (uint256) {
323:    function getSocializingPoolCycleDuration() external view override returns (uint256) {
327:    function getSocializingPoolOptInCoolingPeriod() external view override returns (uint256) {
331:    function getRewardsThreshold() external view override returns (uint256) {
335:    function getMinDepositAmount() external view override returns (uint256) {
339:    function getMaxDepositAmount() external view override returns (uint256) {
343:    function getMinWithdrawAmount() external view override returns (uint256) {
347:    function getMaxWithdrawAmount() external view override returns (uint256) {
351:    function getMinBlockDelayToFinalizeWithdrawRequest() external view override returns (uint256) {
355:    function getWithdrawnKeyBatchSize() external view override returns (uint256) {
361:    function getAdmin() external view returns (address) {
365:    function getStaderTreasury() external view override returns (address) {
371:    function getPoolUtils() external view override returns (address) {
375:    function getPoolSelector() external view override returns (address) {
379:    function getSDCollateral() external view override returns (address) {
383:    function getOperatorRewardsCollector() external view override returns (address) {
387:    function getVaultFactory() external view override returns (address) {
391:    function getStaderOracle() external view override returns (address) {
395:    function getAuctionContract() external view override returns (address) {
399:    function getPenaltyContract() external view override returns (address) {
403:    function getPermissionedPool() external view override returns (address) {
407:    function getStakePoolManager() external view override returns (address) {
411:    function getETHDepositContract() external view override returns (address) {
415:    function getPermissionlessPool() external view override returns (address) {
419:    function getUserWithdrawManager() external view override returns (address) {
423:    function getStaderInsuranceFund() external view override returns (address) {
427:    function getPermissionedNodeRegistry() external view override returns (address) {
431:    function getPermissionlessNodeRegistry() external view override returns (address) {
435:    function getPermissionedSocializingPool() external view override returns (address) {
439:    function getPermissionlessSocializingPool() external view override returns (address) {
443:    function getNodeELRewardVaultImplementation() external view override returns (address) {
447:    function getValidatorWithdrawalVaultImplementation() external view override returns (address) {
452:    function getETHBalancePORFeedProxy() external view override returns (address) {
456:    function getETHXSupplyPORFeedProxy() external view override returns (address) {
462:    function getStaderToken() external view override returns (address) {
466:    function getETHxToken() external view returns (address) {
471:    function setConstant(bytes32 key, uint256 val) internal {
476:    function setVariable(bytes32 key, uint256 val) internal {
481:    function setAccount(bytes32 key, address val) internal {
487:    function setContract(bytes32 key, address val) internal {
493:    function setToken(bytes32 key, address val) internal {
500:    function onlyStaderContract(address _addr, bytes32 _contractName) external view returns (bool) {
504:    function onlyManagerRole(address account) public view override returns (bool) {
508:    function onlyOperatorRole(address account) public view override returns (bool) {
512:    function verifyDepositAndWithdrawLimits() internal view {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol

```solidity
17: contract SocializingPool is
39:    function initialize(address _admin, address _staderConfig) public initializer {
61:    function handleRewards(RewardsData calldata _rewardsData) external override nonReentrant {
107:    function claim(
137:    function _claim(
163:    function verifyProof(
179:    function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {
187:    function getCurrentRewardsIndex() public view returns (uint256 index) {
191:    function getRewardDetails()

// @param missing
206:    function getRewardCycleDetails(uint256 _index) public view returns (uint256 _startBlock, uint256 _endBlock) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol

```solidity
17:   contract SocializingPool is
39:    function initialize(address _admin, address _staderConfig) public initializer {
61:    function handleRewards(RewardsData calldata _rewardsData) external override nonReentrant {
107:    function claim(
137:    function _claim(
163:    function verifyProof(
179:    function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {
187:    function getCurrentRewardsIndex() public view returns (uint256 index) {
191:    function getRewardDetails()

// @return missing
206:    function getRewardCycleDetails(uint256 _index) public view returns (uint256 _startBlock, uint256 _endBlock) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol

```solidity
16:   contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable, ReentrancyGuardUpgradeable {
26:    function initialize(address _admin, address _staderConfig) public initializer {
110:    function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {
119:    function updatePoolThreshold(
193:    function getRewardEligibleSD(address _operator) external view override returns (uint256 _rewardEligibleSD) {
205:    function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {
210:    function convertETHToSD(uint256 _ethAmount) public view override returns (uint256) {
217:    function getOperatorInfo(address _operator)
238:    function isPoolThresholdValid(uint8 _poolId) internal view {

// @param missing
58:    function withdraw(uint256 _requestedSD) external override {
78:    function slashValidatorSD(uint256 _validatorId, uint8 _poolId) external override nonReentrant {
144:    function getOperatorWithdrawThreshold(address _operator) public view returns (uint256 operatorWithdrawThreshold) {

// @return missing
155:    function hasEnoughSDCollateral(
166:    function getMinimumSDToBond(uint8 _poolId, uint256 _numValidator)
183:    function getRemainingSDToBond(

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol

```solidity
12: contract PoolUtils is IPoolUtils, Initializable, AccessControlUpgradeable {
25:    function initialize(address _admin, address _staderConfig) public initializer {
84:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
91:    function getProtocolFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {
157:    function getCollateralETH(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {
162:    function getNodeRegistry(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (address) {
166:    function isExistingPubkey(bytes calldata _pubkey) public view override returns (bool) {
177:    function isExistingOperator(address _operAddr) external view override returns (bool) {
188:    function getOperatorPoolId(address _operAddr) external view override returns (uint8) {
199:    function getValidatorPoolId(bytes calldata _pubkey) external view override returns (uint8) {
210:    function getPoolIdArray() external view override returns (uint8[] memory) {
215:    function onlyValidName(string calldata _name) external view {
225:    function onlyValidKeys(
245:    function calculateRewardShare(uint8 _poolId, uint256 _totalRewards)
271:    function isExistingPoolId(uint8 _poolId) public view override returns (bool) {
281:    function verifyNewPool(uint8 _poolId, address _poolAddress) internal view {
290:    function getPoolCount() internal view returns (uint256) {
295:    modifier onlyExistingPoolId(uint8 _poolId) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol

```solidity
140:    function updatePoolAllocationMaxSize(uint16 _poolAllocationMaxSize) external {
147:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol

```solidity
38:    function initialize(address _admin, address _staderConfig) public initializer {
178:    function getOperatorTotalNonTerminalKeys(
191:    function getCollateralETH() external view override returns (uint256) {
195:    function getNodeRegistry() external view override returns (address) {
200:    function isExistingPubkey(bytes calldata _pubkey) external view override returns (bool) {
205:    function isExistingOperator(address _operAddr) external view override returns (bool) {
210:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
218:    function computeDepositDataRoot(
241:    function fullDepositOnBeaconChain(
273:    function to_little_endian_64(uint256 _depositAmount) internal pure returns (bytes memory ret) {

// @return missing
164:    function getTotalQueuedValidatorCount() external view override returns (uint256) {
171:    function getTotalActiveValidatorCount() external view override returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

```solidity
23: contract PermissionlessNodeRegistry is
66:    function initialize(address _admin, address _staderConfig) public initializer {
287:    function changeSocializingPoolState(bool _optInForSocializingPool)
452:    function getCollateralETH() external pure override returns (uint256) {
589:    function isExistingPubkey(bytes calldata _pubkey) external view override returns (bool) {
594:    function isExistingOperator(address _operAddr) external view override returns (bool) {
598:    function onboardOperator(
618:    function markKeyReadyToDeposit(uint256 _validatorId) internal {
625:    function handleFrontRun(uint256 _validatorId) internal {
633:    function handleInvalidSignature(uint256 _validatorId) internal {
643:    function checkInputKeysCountAndCollateral(
680:    function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {
691:    function isNonTerminalValidator(uint256 _validatorId) internal view returns (bool) {
701:    function isActiveValidator(uint256 _validatorId) internal view returns (bool) {
707:    function decreaseTotalActiveValidatorCount(uint256 _count) internal {
712:    function onlyInitializedValidator(uint256 _validatorId) internal view {
718:    function markValidatorDeposited(uint256 _validatorId) internal {

// @return missing
403:    function getOperatorTotalNonTerminalKeys(
432:    function getOperatorTotalKeys(uint256 _operatorId) public view override returns (uint256 _totalKeys) {
460:    function getOperatorRewardAddress(uint256 _operatorId) external view override returns (address payable) {

// @param missing
526:    function getValidatorsByOperator(
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol

```solidity
21: contract PermissionedPool is IStaderPoolBase, Initializable, AccessControlUpgradeable, ReentrancyGuardUpgradeable {
40:    function initialize(address _admin, address _staderConfig) public initializer {
61:    function receiveInsuranceFund() external payable {
67:    function transferETHOfDefectiveKeysToSSPM(uint256 _defectiveKeyCount) external nonReentrant {
129:    function fullDepositOnBeaconChain(bytes[] calldata _pubkey) external nonReentrant {
199:    function getOperatorTotalNonTerminalKeys(
217:    function getCollateralETH() external view override returns (uint256) {
221:    function getNodeRegistry() external view override returns (address) {
226:    function isExistingPubkey(bytes calldata _pubkey) external view override returns (bool) {
231:    function isExistingOperator(address _operAddr) external view override returns (bool) {
248:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
256:    function computeDepositDataRoot(
279:    function increasePreDepositValidatorCount(uint256 _count) internal {
283:    function decreasePreDepositValidatorCount(uint256 _count) internal {
288:    function preDepositOnBeaconChain(
321:    function to_little_endian_64(uint256 _depositAmount) internal pure returns (bytes memory ret) {

// @return missing
185:     function getTotalQueuedValidatorCount() external view override returns (uint256) {
192:    function getTotalActiveValidatorCount() external view override returns (uint256) {
213:    function getSocializingPoolAddress() external view returns (address) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

```solidity
66:    function initialize(address _admin, address _staderConfig) public initializer {

// @return missing
522:    function getOperatorTotalNonTerminalKeys(
554:    function getOperatorRewardAddress(uint256 _operatorId) external view override returns (address payable) {

// @param missing
620:    function getValidatorsByOperator(
656:    function onlyPreDepositValidator(bytes calldata _pubkey) external view override {
671:    function handleFrontRun(uint256 _validatorId) internal {
680:    function getOperatorQueuedValidatorCount(uint256 _operatorId) internal view returns (uint256 _validatorCount) {

546:    function getCollateralETH() external pure override returns (uint256) {
646:    function isExistingPubkey(bytes calldata _pubkey) external view override returns (bool) {
651:    function isExistingOperator(address _operAddr) external view override returns (bool) {
661:    function onboardOperator(string calldata _operatorName, address payable _operatorRewardAddress) internal {
687:    function checkInputKeysCountAndCollateral(
720:    function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {
732:    function isActiveValidator(uint256 _validatorId) internal view returns (bool) {
738:    function isNonTerminalValidator(uint256 _validatorId) internal view returns (bool) {
747:    function decreaseTotalActiveValidatorCount(uint256 _count) internal {
752:    function onlyPreDepositValidator(uint256 _validatorId) internal view {
758:    function markValidatorDeposited(uint256 _validatorId) internal {
762:    function _deactivateNodeOperator(uint256 _operatorId) internal {
768:    function _activateNodeOperator(uint256 _operatorId) internal {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol

```solidity
14:   contract Penalty is IPenalty, Initializable, AccessControlUpgradeable, ReentrancyGuardUpgradeable {
31:    function initialize(
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol

```solidity
12: contract OperatorRewardsCollector is
27:    function initialize(address _admin, address _staderConfig) external initializer {
40:    function depositFor(address _receiver) external payable {
46:    function claim() external whenNotPaused {
56:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol

```solidity
13: contract NodeELRewardVault is INodeELRewardVault {
24:    function withdraw() external override {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol

```solidity
29:     function initialize(address _admin, address _staderConfig) public initializer {
77:    function _beforeTokenTransfer(
85:    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol

```solidity
14: contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableUpgradeable, ReentrancyGuardUpgradeable {
29:    function initialize(address _admin, address _staderConfig) external initializer {
48:    function createLot(uint256 _sdAmount) external override whenNotPaused {
62:    function addBid(uint256 lotId) external payable override whenNotPaused {
80:    function claimSD(uint256 lotId) external override {
93:    function transferHighestBidToSSPM(uint256 lotId) external override nonReentrant {
106:    function extractNonBidSD(uint256 lotId) external override {
120:    function withdrawUnselectedBid(uint256 lotId) external override nonReentrant {
138:    function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {
144:    function updateDuration(uint256 _duration) external override {
151:    function updateBidIncrement(uint256 _bidIncrement) external override {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
21:    function checkNonZeroAddress(address _address) internal pure {
26:    function onlyManagerRole(address _addr, IStaderConfig _staderConfig) internal view {
32:    function onlyOperatorRole(address _addr, IStaderConfig _staderConfig) internal view {
39:    function onlyStaderContract(
49:    function getPubkeyForValidSender(
65:    function getOperatorForValidSender(
82:    function onlyValidatorWithdrawVault(
95:    function getOperatorAddressByValidatorId(
107:    function getOperatorAddressByOperatorId(
118:    function getOperatorRewardAddress(address _operator, IStaderConfig _staderConfig)
134:    function getPubkeyRoot(bytes calldata _pubkey) internal pure returns (bytes32) {
143:    function getValidatorSettleStatus(bytes calldata _pubkey, IStaderConfig _staderConfig)
155:    function computeExchangeRate(
167:    function sendValue(address _receiver, uint256 _amount) internal {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol

```solidity
// private and internal `variables` should preppend with `underline
19:    bool internal vaultSettleStatus;

// private and internal `functions` should preppend with `underline`
127:    function getCollateralETH(uint8 _poolId, IStaderConfig _staderConfig) internal view returns (uint256) {
131:    function getOperatorAddress(
139:    function getUpdatedPenaltyAmount(
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol

```soldity
206:    function deleteRequestId(uint256 _requestId, address _owner) internal {
223:    function sendValue(address payable _recipient, uint256 _amount) internal {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol

```solidity
//  private and internal `functions` should preppend with `underline`
284:    function initialConvertToShares(
305:    function initialConvertToAssets(
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

```solidity
// private and internal `variables` should preppend with `underline`
45:    mapping(bytes32 => bool) private nodeSubmissionKeys;
46:    mapping(bytes32 => uint8) private submissionCountKeys;
48:    uint256[] private sdPrices;

// private and internal `functions` should preppend with `underline`
300:    function insertSDPrice(uint256 _sdPrice) internal {
312:    function getMedianValue(uint256[] storage dataArray) internal view returns (uint256 _medianValue) {
559:    function setUpdateFrequency(bytes32 _key, uint256 _updateFrequency) internal {
598:    function getReportableBlockFor(bytes32 _key) internal view returns (uint256) {
620:    function attestSubmission(bytes32 _nodeSubmissionKey, bytes32 _submissionCountKey)
637:    function getPORFeedData()
653:    function updateWithInLimitER(
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol

```solidity
// private and internal `functions` should preppend with `underline`
471:    function setConstant(bytes32 key, uint256 val) internal {
476:    function setVariable(bytes32 key, uint256 val) internal {
481:    function setAccount(bytes32 key, address val) internal {
487:    function setContract(bytes32 key, address val) internal {
493:    function setToken(bytes32 key, address val) internal {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol

```solidity
// private and internal `variables` should preppend with `underline`
74:    mapping(bytes32 => uint256) private constantsMap;
75:    mapping(bytes32 => uint256) private variablesMap;
76:    mapping(bytes32 => address) private accountsMap;
77:    mapping(bytes32 => address) private contractsMap;
78:    mapping(bytes32 => address) private tokensMap;
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol

```solidity
// private and internal `functions` should preppend with `underline`
90:    function slashSD(address _operator, uint256 _sdToSlash) internal {
217:    function getOperatorInfo(address _operator)
238:    function isPoolThresholdValid(uint8 _poolId) internal view {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol

```solidity
// private and internal `functions` should preppend with `underline`
281:    function verifyNewPool(uint8 _poolId, address _poolAddress) internal view {
290:    function getPoolCount() internal view returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol

```solidity
// private and internal `functions` should preppend with `underline`
241:    function fullDepositOnBeaconChain(
273:    function to_little_endian_64(uint256 _depositAmount) internal pure returns (bytes memory ret) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

```solidity
// private and internal `functions` should preppend with `underline`
598:    function onboardOperator(
618:    function markKeyReadyToDeposit(uint256 _validatorId) internal {
625:    function handleFrontRun(uint256 _validatorId) internal {
633:    function handleInvalidSignature(uint256 _validatorId) internal {
643:    function checkInputKeysCountAndCollateral(
680:    function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {
691:    function isNonTerminalValidator(uint256 _validatorId) internal view returns (bool) {
701:    function isActiveValidator(uint256 _validatorId) internal view returns (bool) {
707:    function decreaseTotalActiveValidatorCount(uint256 _count) internal {
712:    function onlyInitializedValidator(uint256 _validatorId) internal view {
718:    function markValidatorDeposited(uint256 _validatorId) internal {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol

```solidity
279:    function increasePreDepositValidatorCount(uint256 _count) internal {
283:    function decreasePreDepositValidatorCount(uint256 _count) internal {
288:    function preDepositOnBeaconChain(
321:    function to_little_endian_64(uint256 _depositAmount) internal pure returns (bytes memory ret) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

```solidity
// external non-view coming before external views
475:    function increaseTotalActiveValidatorCount(uint256 _count) external override {

// external pure/view functions should come after all the non-view/non-pure external functions
546:     function getCollateralETH() external pure override returns (uint256) {
554:    function getOperatorRewardAddress(uint256 _operatorId) external view override returns (address payable) {

// public functions coming after external ones
513:    function getOperatorTotalKeys(uint256 _operatorId) public view override returns (uint256 _totalKeys) {
522:    function getOperatorTotalNonTerminalKeys(

// internal view/pure functions coming before non-view/non-pure internal functions
680:    function getOperatorQueuedValidatorCount(uint256 _operatorId) internal view returns (uint256 _validatorCount) {
687:    function checkInputKeysCountAndCollateral(
720:    function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {
732:    function isActiveValidator(uint256 _validatorId) internal view returns (bool) {
738:    function isNonTerminalValidator(uint256 _validatorId) internal view returns (bool) {
752:    function onlyPreDepositValidator(uint256 _validatorId) internal view {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

```solidity
661:    function onboardOperator(string calldata _operatorName, address payable _operatorRewardAddress) internal {
671:    function handleFrontRun(uint256 _validatorId) internal {
680:    function getOperatorQueuedValidatorCount(uint256 _operatorId) internal view returns (uint256 _validatorCount) {
687:    function checkInputKeysCountAndCollateral(
720:    function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {
732:    function isActiveValidator(uint256 _validatorId) internal view returns (bool) {
738:    function isNonTerminalValidator(uint256 _validatorId) internal view returns (bool) {
747:    function decreaseTotalActiveValidatorCount(uint256 _count) internal {
752:    function onlyPreDepositValidator(uint256 _validatorId) internal view {
758:    function markValidatorDeposited(uint256 _validatorId) internal {
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol

```solidity
// lose the ^ for a safer code
2:  pragma solidity ^0.8.16;
```

---

### Initialized variables [6]

- Variables are default initialized with 0 for `uint / int`, 0x0 for `address` and false for `bool`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol

```solidity
232:        for (uint256 i = 0; i < poolCount; i++) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol

```solidity
145:        for (uint256 i = 0; i < indexLength; i++) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol

```solidity
//  initializing uint with default value 0
145:        for (uint256 i = 0; i < indexLength; i++) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol

```solidity
//  initializing uint with default value 0
104:        for (uint256 i = 0; i < poolCount; i++) {
168:        for (uint256 i = 0; i < poolCount; i++) {
179:        for (uint256 i = 0; i < poolCount; i++) {
190:        for (uint256 i = 0; i < poolCount; i++) {
201:        for (uint256 i = 0; i < poolCount; i++) {
273:        for (uint256 i = 0; i < poolCount; i++) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol

```solidity
//  initializing uint with default value 0
130:        for (uint256 i = 0; i < poolTargetLength; i++) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

```solidity
//  initializing uint with default value 0
503:        uint256 validatorCount = 0;
572:        uint256 optOutOperatorCount = 0;
```

---

### Observations [7]

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol

```solidity
// add a 2 step for changin the _staderConfig address in case is not set to the expected address
86:    function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol

```solidity
// add a 2 step for changin the _staderConfig address in case is not set to the expected address
57:    function updateStaderConfig(address _staderConfig) external override onlyOwner {

// add a 2 step for changing the owner in case _owner is not set to the expected address
68:    function updateOwner(address _owner) external override onlyOwner {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol

```solidity
// half of function params are being defined with underline prefixed, choose a pattern and stick to it.
```

