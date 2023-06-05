# VULN 1 

## [GAS] Generate merkle tree offchain
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 173 at 2023-06-stader-labs/contracts/SocializingPool.sol:

        bytes32 merkleRoot = rewardsDataMap[_index].merkleRoot;

------------------------------------------------------------------------ 

### Mitigation 

Generating merkle tree onchain is expensive, especially when you want to include a large set of values. Consider generating it offchain and publishing the root when creating a new pool.










# VULN 2 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 115 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        if (totalRewards > 0) {


Found in line 330 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

            (totalAssets() > 0 ||


Found in line 109 at 2023-06-stader-labs/contracts/Auction.sol:

        if (lotItem.highestBidAmount > 0) revert LotWasAuctioned();


Found in line 119 at 2023-06-stader-labs/contracts/SocializingPool.sol:

        if (totalAmountETH > 0) {


Found in line 127 at 2023-06-stader-labs/contracts/SocializingPool.sol:

        if (totalAmountSD > 0) {


Found in line 204 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);


Found in line 299 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        if (totalDefectedKeys > 0) {


Found in line 209 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

        if (frontRunValidatorsLength > 0) {


Found in line 305 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

            if (address(feeRecipientAddress).balance > 0) {


Found in line 121 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        if (_exchangeRate.reportingBlockNumber % updateFrequencyMap[ETHX_ER_UF] > 0) {


Found in line 274 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        if (_sdPriceData.reportingBlockNumber % updateFrequencyMap[SD_PRICE_UF] > 0) {


Found in line 328 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        if (_validatorStats.reportingBlockNumber % updateFrequencyMap[VALIDATOR_STATS_UF] > 0) {


Found in line 403 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        if (_withdrawnValidators.reportingBlockNumber % updateFrequencyMap[WITHDRAWN_VALIDATORS_UF] > 0) {

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 3 

## [GAS] Use shift Right/Left instead of division/multiplication if possible
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 148 at 2023-06-stader-labs/contracts/StaderOracle.sol:

            submissionCount == trustedNodesCount / 2 + 1 &&


Found in line 255 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        if ((submissionCount == trustedNodesCount / 2 + 1)) {


Found in line 290 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        if ((submissionCount == (2 * trustedNodesCount) / 3 + 1)) {


Found in line 314 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        return (dataArray[(len - 1) / 2] + dataArray[len / 2]) / 2;


Found in line 372 at 2023-06-stader-labs/contracts/StaderOracle.sol:

            submissionCount == trustedNodesCount / 2 + 1 &&


Found in line 432 at 2023-06-stader-labs/contracts/StaderOracle.sol:

            submissionCount == trustedNodesCount / 2 + 1 &&


Found in line 482 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        if ((submissionCount == trustedNodesCount / 2 + 1)) {

------------------------------------------------------------------------ 

### Mitigation 

Use shift Right/Left instead of division/multiplication if possible.










# VULN 4 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 232 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 59 at 2023-06-stader-labs/contracts/Auction.sol:

        nextLot++;


Found in line 145 at 2023-06-stader-labs/contracts/SocializingPool.sol:

        for (uint256 i = 0; i < indexLength; i++) {


Found in line 130 at 2023-06-stader-labs/contracts/PoolSelector.sol:

        for (uint256 i = 0; i < poolTargetLength; i++) {


Found in line 91 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        for (uint256 i = 0; i < permissionedNosLength; i++) {


Found in line 176 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

            nextValidatorId++;


Found in line 537 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

                totalNonWithdrawnKeyCount++;


Found in line 598 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        for (uint256 i = startIndex; i < endIndex; i++) {


Found in line 601 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

                validatorCount++;


Found in line 637 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        for (uint256 i = startIndex; i < endIndex; i++) {


Found in line 665 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        nextOperatorId++;


Found in line 666 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        totalActiveOperatorCount++;


Found in line 770 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        totalActiveOperatorCount++;


Found in line 162 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

            nextValidatorId++;


Found in line 418 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

                totalNonWithdrawnKeyCount++;


Found in line 504 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

        for (uint256 i = startIndex; i < endIndex; i++) {


Found in line 507 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

                validatorCount++;


Found in line 543 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

        for (uint256 i = startIndex; i < endIndex; i++) {


Found in line 573 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

        for (uint256 i = startIndex; i < endIndex; i++) {


Found in line 576 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

                optOutOperatorCount++;


Found in line 612 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

        nextOperatorId++;


Found in line 621 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

        validatorQueueSize++;


Found in line 104 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 168 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 179 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 190 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 201 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 273 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 88 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        trustedNodesCount++;


Found in line 487 at 2023-06-stader-labs/contracts/StaderOracle.sol:

                missedAttestationPenalty[pubkeyRoot]++;


Found in line 629 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        submissionCountKeys[_submissionCountKey]++;


Found in line 109 at 2023-06-stader-labs/contracts/UserWithdrawalManager.sol:

        nextRequestId++;

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 5 

## [GAS] Don’t initialize variables with default value
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 232 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 145 at 2023-06-stader-labs/contracts/SocializingPool.sol:

        for (uint256 i = 0; i < indexLength; i++) {


Found in line 130 at 2023-06-stader-labs/contracts/PoolSelector.sol:

        for (uint256 i = 0; i < poolTargetLength; i++) {


Found in line 91 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        for (uint256 i = 0; i < permissionedNosLength; i++) {


Found in line 597 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        uint256 validatorCount = 0;


Found in line 503 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

        uint256 validatorCount = 0;


Found in line 572 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

        uint256 optOutOperatorCount = 0;


Found in line 104 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 168 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 179 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 190 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 201 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {


Found in line 273 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        for (uint256 i = 0; i < poolCount; i++) {

------------------------------------------------------------------------ 

### Mitigation 

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.










# VULN 6 

## [GAS] Use assembly to check for address(0)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 96 at 2023-06-stader-labs/contracts/UserWithdrawalManager.sol:

        if (_owner == address(0)) revert ZeroAddressReceived();


Found in line 22 at 2023-06-stader-labs/contracts/library/UtilLib.sol:

        if (_address == address(0)) revert ZeroAddress();

------------------------------------------------------------------------ 

### Mitigation 

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression, especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.










# VULN 7 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 31 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint8 poolId = VaultProxy(payable(address(this))).poolId();


Found in line 55 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint8 poolId = VaultProxy(payable(address(this))).poolId();


Found in line 94 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint8 poolId = VaultProxy(payable(address(this))).poolId();


Found in line 127 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

    function getCollateralETH(uint8 _poolId, IStaderConfig _staderConfig) internal view returns (uint256) {


Found in line 132 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint8 _poolId,


Found in line 140 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint8 _poolId,


Found in line 90 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function receiveExcessEthFromPool(uint8 _poolId) external payable override {


Found in line 183 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function validatorBatchDeposit(uint8 _poolId) external override nonReentrant whenNotPaused {


Found in line 227 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

        (uint256[] memory selectedPoolCapacity, uint8[] memory poolIdArray) = IPoolSelector(


Found in line 18 at 2023-06-stader-labs/contracts/PoolSelector.sol:

    uint16 public poolAllocationMaxSize;


Found in line 23 at 2023-06-stader-labs/contracts/PoolSelector.sol:

    mapping(uint8 => uint256) public poolWeights;


Found in line 50 at 2023-06-stader-labs/contracts/PoolSelector.sol:

    function computePoolAllocationForDeposit(uint8 _poolId, uint256 _newValidatorToRegister)


Found in line 79 at 2023-06-stader-labs/contracts/PoolSelector.sol:

        returns (uint256[] memory selectedPoolCapacity, uint8[] memory poolIdArray)


Found in line 121 at 2023-06-stader-labs/contracts/PoolSelector.sol:

        uint8[] memory poolIdArray = IPoolUtils(staderConfig.getPoolUtils()).getPoolIdArray();


Found in line 140 at 2023-06-stader-labs/contracts/PoolSelector.sol:

    function updatePoolAllocationMaxSize(uint16 _poolAllocationMaxSize) external {


Found in line 144 at 2023-06-stader-labs/contracts/Penalty.sol:

    function markValidatorSettled(uint8 _poolId, uint256 _validatorId) external override {


Found in line 12 at 2023-06-stader-labs/contracts/VaultProxy.sol:

    uint8 public override poolId;


Found in line 22 at 2023-06-stader-labs/contracts/VaultProxy.sol:

        uint8 _poolId,


Found in line 31 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

    uint8 public constant override POOL_ID = 2;


Found in line 32 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

    uint16 public override inputKeyCountLimit;


Found in line 427 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

    function updateInputKeyCountLimit(uint16 _inputKeyCountLimit) external override {


Found in line 25 at 2023-06-stader-labs/contracts/NodeELRewardVault.sol:

        uint8 poolId = IVaultProxy(address(this)).poolId();


Found in line 30 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

    uint8 public constant override POOL_ID = 1;


Found in line 31 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

    uint16 public override inputKeyCountLimit;


Found in line 325 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

    function updateInputKeyCountLimit(uint16 _inputKeyCountLimit) external override {


Found in line 18 at 2023-06-stader-labs/contracts/SDCollateral.sol:

    mapping(uint8 => PoolThresholdInfo) public poolThresholdbyPoolId;


Found in line 78 at 2023-06-stader-labs/contracts/SDCollateral.sol:

    function slashValidatorSD(uint256 _validatorId, uint8 _poolId) external override nonReentrant {


Found in line 120 at 2023-06-stader-labs/contracts/SDCollateral.sol:

        uint8 _poolId,


Found in line 145 at 2023-06-stader-labs/contracts/SDCollateral.sol:

        (uint8 poolId, , uint256 validatorCount) = getOperatorInfo(_operator);


Found in line 157 at 2023-06-stader-labs/contracts/SDCollateral.sol:

        uint8 _poolId,


Found in line 166 at 2023-06-stader-labs/contracts/SDCollateral.sol:

    function getMinimumSDToBond(uint8 _poolId, uint256 _numValidator)


Found in line 185 at 2023-06-stader-labs/contracts/SDCollateral.sol:

        uint8 _poolId,


Found in line 194 at 2023-06-stader-labs/contracts/SDCollateral.sol:

        (uint8 poolId, , uint256 validatorCount) = getOperatorInfo(_operator);


Found in line 221 at 2023-06-stader-labs/contracts/SDCollateral.sol:

            uint8 _poolId,


Found in line 238 at 2023-06-stader-labs/contracts/SDCollateral.sol:

    function isPoolThresholdValid(uint8 _poolId) internal view {


Found in line 17 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    mapping(uint8 => address) public override poolAddressById;


Found in line 18 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    uint8[] public override poolIdArray;


Found in line 40 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function addNewPool(uint8 _poolId, address _poolAddress) external override {


Found in line 55 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function updatePoolAddress(uint8 _poolId, address _newPoolAddress)


Found in line 91 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getProtocolFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {


Found in line 96 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getOperatorFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {


Found in line 112 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getQueuedValidatorCountByPool(uint8 _poolId)


Found in line 124 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getActiveValidatorCountByPool(uint8 _poolId)


Found in line 136 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getSocializingPoolAddress(uint8 _poolId)


Found in line 148 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        uint8 _poolId,


Found in line 157 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getCollateralETH(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {


Found in line 162 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getNodeRegistry(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (address) {


Found in line 188 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getOperatorPoolId(address _operAddr) external view override returns (uint8) {


Found in line 199 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getValidatorPoolId(bytes calldata _pubkey) external view override returns (uint8) {


Found in line 210 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getPoolIdArray() external view override returns (uint8[] memory) {


Found in line 245 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function calculateRewardShare(uint8 _poolId, uint256 _totalRewards)


Found in line 271 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function isExistingPoolId(uint8 _poolId) public view override returns (bool) {


Found in line 281 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function verifyNewPool(uint8 _poolId, address _poolAddress) internal view {


Found in line 295 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    modifier onlyExistingPoolId(uint8 _poolId) {


Found in line 46 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    mapping(bytes32 => uint8) private submissionCountKeys;


Found in line 47 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    mapping(bytes32 => uint16) public override missedAttestationPenalty;


Found in line 137 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);


Found in line 253 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);


Found in line 284 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);


Found in line 357 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);


Found in line 421 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);


Found in line 471 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        uint8 submissionCount = attestSubmission(nodeSubmissionKey, submissionCountKey);


Found in line 575 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    function getMerkleRootReportableBlockByPoolId(uint8 _poolId) public view override returns (uint256) {


Found in line 606 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    function getCurrentRewardsIndexByPoolId(uint8 _poolId) public view returns (uint256) {


Found in line 622 at 2023-06-stader-labs/contracts/StaderOracle.sol:

        returns (uint8 _submissionCount)


Found in line 35 at 2023-06-stader-labs/contracts/factory/VaultFactory.sol:

        uint8 _poolId,


Found in line 48 at 2023-06-stader-labs/contracts/factory/VaultFactory.sol:

    function deployNodeELRewardVault(uint8 _poolId, uint256 _operatorId)


Found in line 63 at 2023-06-stader-labs/contracts/factory/VaultFactory.sol:

        uint8 _poolId,


Found in line 71 at 2023-06-stader-labs/contracts/factory/VaultFactory.sol:

    function computeNodeELRewardVaultAddress(uint8 _poolId, uint256 _operatorId)


Found in line 50 at 2023-06-stader-labs/contracts/library/UtilLib.sol:

        uint8 _poolId,


Found in line 66 at 2023-06-stader-labs/contracts/library/UtilLib.sol:

        uint8 _poolId,


Found in line 83 at 2023-06-stader-labs/contracts/library/UtilLib.sol:

        uint8 _poolId,


Found in line 96 at 2023-06-stader-labs/contracts/library/UtilLib.sol:

        uint8 _poolId,


Found in line 108 at 2023-06-stader-labs/contracts/library/UtilLib.sol:

        uint8 _poolId,


Found in line 123 at 2023-06-stader-labs/contracts/library/UtilLib.sol:

        uint8 poolId = IPoolUtils(_staderConfig.getPoolUtils()).getOperatorPoolId(_operator);


Found in line 148 at 2023-06-stader-labs/contracts/library/UtilLib.sol:

        uint8 poolId = IPoolUtils(_staderConfig.getPoolUtils()).getValidatorPoolId(_pubkey);

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 8 

## [GAS] Public Functions to external
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 125 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function getExchangeRate() public view override returns (uint256) {


Found in line 135 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function totalAssets() public view override returns (uint256) {


Found in line 140 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function convertToShares(uint256 _assets) public view override returns (uint256) {


Found in line 145 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function convertToAssets(uint256 _shares) public view override returns (uint256) {


Found in line 150 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function maxDeposit() public view override returns (uint256) {


Found in line 154 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function minDeposit() public view override returns (uint256) {


Found in line 159 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function previewDeposit(uint256 _assets) public view override returns (uint256) {


Found in line 164 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function previewWithdraw(uint256 _shares) public view override returns (uint256) {


Found in line 169 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function deposit(address _receiver) public payable override whenNotPaused returns (uint256) {


Found in line 328 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    function isVaultHealthy() public view override returns (bool) {


Found in line 504 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    function onlyManagerRole(address account) public view override returns (bool) {


Found in line 508 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    function onlyOperatorRole(address account) public view override returns (bool) {


Found in line 123 at 2023-06-stader-labs/contracts/Penalty.sol:

    function calculateMEVTheftPenalty(bytes32 _pubkeyRoot) public override returns (uint256) {


Found in line 132 at 2023-06-stader-labs/contracts/Penalty.sol:

    function calculateMissedAttestationPenalty(bytes32 _pubkeyRoot) public view override returns (uint256) {


Found in line 513 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

    function getOperatorTotalKeys(uint256 _operatorId) public view override returns (uint256 _totalKeys) {


Found in line 432 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

    function getOperatorTotalKeys(uint256 _operatorId) public view override returns (uint256 _totalKeys) {


Found in line 440 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

    function getTotalQueuedValidatorCount() public view override returns (uint256) {


Found in line 205 at 2023-06-stader-labs/contracts/SDCollateral.sol:

    function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {


Found in line 210 at 2023-06-stader-labs/contracts/SDCollateral.sol:

    function convertETHToSD(uint256 _ethAmount) public view override returns (uint256) {


Found in line 91 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getProtocolFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {


Found in line 96 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getOperatorFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {


Found in line 157 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getCollateralETH(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {


Found in line 162 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function getNodeRegistry(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (address) {


Found in line 166 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function isExistingPubkey(bytes calldata _pubkey) public view override returns (bool) {


Found in line 271 at 2023-06-stader-labs/contracts/PoolUtils.sol:

    function isExistingPoolId(uint8 _poolId) public view override returns (bool) {


Found in line 185 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    function disableERInspectionMode() public override whenNotPaused {


Found in line 571 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    function getERReportableBlock() public view override returns (uint256) {


Found in line 575 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    function getMerkleRootReportableBlockByPoolId(uint8 _poolId) public view override returns (uint256) {


Found in line 582 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    function getSDPriceReportableBlock() public view override returns (uint256) {


Found in line 586 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    function getValidatorStatsReportableBlock() public view override returns (uint256) {


Found in line 590 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    function getWithdrawnValidatorReportableBlock() public view override returns (uint256) {


Found in line 594 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    function getMissedAttestationPenaltyReportableBlock() public view override returns (uint256) {

------------------------------------------------------------------------ 

### Mitigation 

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.










# VULN 9 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 280 at 2023-06-stader-labs/contracts/PermissionedPool.sol:

        preDepositValidatorCount += _count;


Found in line 119 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

            _userShare += userReward;


Found in line 120 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

            _operatorShare += operatorReward;


Found in line 121 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

            _protocolShare += protocolReward;


Found in line 81 at 2023-06-stader-labs/contracts/SocializingPool.sol:

        totalOperatorETHRewardsRemaining += _rewardsData.operatorETHRewards;


Found in line 82 at 2023-06-stader-labs/contracts/SocializingPool.sol:

        totalOperatorSDRewardsRemaining += _rewardsData.operatorSDRewards;


Found in line 153 at 2023-06-stader-labs/contracts/SocializingPool.sol:

            _totalAmountSD += _amountSD[i];


Found in line 154 at 2023-06-stader-labs/contracts/SocializingPool.sol:

            _totalAmountETH += _amountETH[i];


Found in line 98 at 2023-06-stader-labs/contracts/PoolSelector.sol:

            selectedValidatorCount += selectedPoolCapacity[i];


Found in line 131 at 2023-06-stader-labs/contracts/PoolSelector.sol:

            totalWeight += _poolTargets[i];


Found in line 56 at 2023-06-stader-labs/contracts/Penalty.sol:

        additionalPenaltyAmount[pubkeyRoot] += _amount;


Found in line 212 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

                totalValidatorToDeposit += selectedOperatorCapacity[i];


Found in line 234 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

                selectedOperatorCapacity[i] += newSelectedCapacity;


Found in line 477 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

        totalActiveValidatorCount += _count;


Found in line 490 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

                totalQueuedValidators += getOperatorQueuedValidatorCount(i);


Found in line 41 at 2023-06-stader-labs/contracts/OperatorRewardsCollector.sol:

        balances[_receiver] += msg.value;


Found in line 383 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

        totalActiveValidatorCount += _count;


Found in line 45 at 2023-06-stader-labs/contracts/SDCollateral.sol:

        operatorSDBalance[operator] += _sdAmount;


Found in line 105 at 2023-06-stader-labs/contracts/PoolUtils.sol:

            totalActiveValidatorCount += getActiveValidatorCountByPool(poolIdArray[i]);


Found in line 266 at 2023-06-stader-labs/contracts/PoolUtils.sol:

        operatorShare += (operatorFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();


Found in line 105 at 2023-06-stader-labs/contracts/UserWithdrawalManager.sol:

        ethRequestedForWithdraw += assets;


Found in line 146 at 2023-06-stader-labs/contracts/UserWithdrawalManager.sol:

            lockedEthXToBurn += lockedEthX;


Found in line 147 at 2023-06-stader-labs/contracts/UserWithdrawalManager.sol:

            ethToSendToFinalizeRequest += minEThRequiredToFinalizeRequest;

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 10 

## [GAS] Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 29 and 30 at 2023-06-stader-labs/contracts/SocializingPool.sol:

    mapping(address => mapping(uint256 => bool)) public override claimedRewards;

    mapping(uint256 => bool) public handledRewards;


Found in line 74 and 75 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    mapping(bytes32 => uint256) private constantsMap;

    mapping(bytes32 => uint256) private variablesMap;


Found in line 76 and 77 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    mapping(bytes32 => address) private accountsMap;

    mapping(bytes32 => address) private contractsMap;


Found in line 23 and 24 at 2023-06-stader-labs/contracts/Penalty.sol:

    /// @inheritdoc IPenalty

    mapping(bytes => uint256) public override totalPenaltyAmount;


Found in line 47 and 48 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

    // mapping of bytes public key and validator Id

    mapping(bytes => uint256) public override validatorIdByPubkey;


Found in line 51 and 52 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

    // mapping of operator address and operator Id

    mapping(address => uint256) public override operatorIDByAddress;


Found in line 55 and 56 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

    //mapping of operator wise queued validator Ids arrays

    mapping(uint256 => uint256[]) public override validatorIdsByOperatorId;


Found in line 58 and 59 at 2023-06-stader-labs/contracts/PermissionedNodeRegistry.sol:

    mapping(uint256 => uint256) public override nextQueuedValidatorIndexByOperatorId;

    mapping(uint256 => uint256) public socializingPoolStateChangeBlock;


Found in line 47 and 48 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

    // mapping of validator public key and validator Id

    mapping(bytes => uint256) public override validatorIdByPubkey;


Found in line 51 and 52 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

    // mapping of operator Id and Operator struct

    mapping(uint256 => Operator) public override operatorStructById;


Found in line 55 and 56 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

    //mapping of operator wise validator Ids arrays

    mapping(uint256 => uint256[]) public override validatorIdsByOperatorId;


Found in line 58 and 59 at 2023-06-stader-labs/contracts/PermissionlessNodeRegistry.sol:

    //mapping of operator address with nodeELReward vault address

    mapping(uint256 => address) public override nodeELRewardVaultByOperatorId;


Found in line 18 and 19 at 2023-06-stader-labs/contracts/SDCollateral.sol:

    mapping(uint8 => PoolThresholdInfo) public poolThresholdbyPoolId;

    mapping(address => uint256) public override operatorSDBalance;


Found in line 44 and 45 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    mapping(address => bool) public override isTrustedNode;

    mapping(bytes32 => bool) private nodeSubmissionKeys;


Found in line 46 and 47 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    mapping(bytes32 => uint8) private submissionCountKeys;

    mapping(bytes32 => uint16) public override missedAttestationPenalty;

------------------------------------------------------------------------ 

### Mitigation 

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.










# VULN 11 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 174 at 2023-06-stader-labs/contracts/PermissionedPool.sol:

        if (preDepositValidatorCount != 0 || address(this).balance == 0) {


Found in line 178 at 2023-06-stader-labs/contracts/PermissionedPool.sol:

            value: address(this).balance


Found in line 31 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint8 poolId = VaultProxy(payable(address(this))).poolId();


Found in line 32 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint256 validatorId = VaultProxy(payable(address(this))).id();


Found in line 33 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();


Found in line 34 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint256 totalRewards = address(this).balance;


Found in line 55 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint8 poolId = VaultProxy(payable(address(this))).poolId();


Found in line 56 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint256 validatorId = VaultProxy(payable(address(this))).id();


Found in line 57 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();


Found in line 94 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint8 poolId = VaultProxy(payable(address(this))).poolId();


Found in line 95 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();


Found in line 99 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

        uint256 contractBalance = address(this).balance;


Found in line 189 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

            address(this).balance,


Found in line 221 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

            address(this).balance,


Found in line 55 at 2023-06-stader-labs/contracts/Auction.sol:

        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {


Found in line 69 at 2023-06-stader-labs/contracts/SocializingPool.sol:

            address(this).balance - totalOperatorETHRewardsRemaining


Found in line 75 at 2023-06-stader-labs/contracts/SocializingPool.sol:

            IERC20(staderConfig.getStaderToken()).balanceOf(address(this)) - totalOperatorSDRewardsRemaining


Found in line 25 at 2023-06-stader-labs/contracts/NodeELRewardVault.sol:

        uint8 poolId = IVaultProxy(address(this)).poolId();


Found in line 26 at 2023-06-stader-labs/contracts/NodeELRewardVault.sol:

        uint256 operatorId = IVaultProxy(address(this)).id();


Found in line 27 at 2023-06-stader-labs/contracts/NodeELRewardVault.sol:

        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();


Found in line 28 at 2023-06-stader-labs/contracts/NodeELRewardVault.sol:

        uint256 totalRewards = address(this).balance;


Found in line 47 at 2023-06-stader-labs/contracts/SDCollateral.sol:

        if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {


Found in line 43 at 2023-06-stader-labs/contracts/StaderInsuranceFund.sol:

        if (address(this).balance < _amount || _amount == 0) {


Found in line 62 at 2023-06-stader-labs/contracts/StaderInsuranceFund.sol:

        if (address(this).balance < _amount) {


Found in line 104 at 2023-06-stader-labs/contracts/UserWithdrawalManager.sol:

        IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);


Found in line 155 at 2023-06-stader-labs/contracts/UserWithdrawalManager.sol:

            ETHx(staderConfig.getETHxToken()).burnFrom(address(this), lockedEthXToBurn);


Found in line 224 at 2023-06-stader-labs/contracts/UserWithdrawalManager.sol:

        if (address(this).balance < _amount) {

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 12 

## [GAS] Functions guaranteed to revert when called by normal users can be marked payable
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 57 at 2023-06-stader-labs/contracts/VaultProxy.sol:

    function updateStaderConfig(address _staderConfig) external override onlyOwner {


Found in line 68 at 2023-06-stader-labs/contracts/VaultProxy.sol:

    function updateOwner(address _owner) external override onlyOwner {

------------------------------------------------------------------------ 

### Mitigation 

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.










# VULN 13 

## [GAS] Do not calculate constants
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 22 at 2023-06-stader-labs/contracts/Auction.sol:

    uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours


Found in line 26 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days


Found in line 50 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency


Found in line 51 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key


Found in line 52 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key


Found in line 53 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key


Found in line 54 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key

------------------------------------------------------------------------ 

### Mitigation 

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.










# VULN 14 

## [GAS] >= costs less gas than >
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 202 at 2023-06-stader-labs/contracts/SDCollateral.sol:

        return (sdBalance < totalMinThreshold ? 0 : Math.min(sdBalance, totalMaxThreshold));

------------------------------------------------------------------------ 

### Mitigation 

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.










# VULN 15 

## [GAS] Change public state variable visibility to private
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 16 at 2023-06-stader-labs/contracts/Penalty.sol:

    address public override ratedOracleAddress;


Found in line 14 at 2023-06-stader-labs/contracts/VaultProxy.sol:

    address public override owner;

------------------------------------------------------------------------ 

### Mitigation 

It is preferred to change the visibility of the state variables to private, this will save significant gas.










# VULN 16 

## [GAS] With assembly, .call (bool success) transfer can be done gas-optimized
------------------------------------------------------------------------ 

### Proof of concept 

Found in 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol (function transferETHToUserWithdrawManager(uint256 _amount) external override nonReentrant whenNotPaused {):

     * @param _amount amount of ETH to transfer

     */

    function transferETHToUserWithdrawManager(uint256 _amount) external override nonReentrant whenNotPaused {

        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.USER_WITHDRAW_MANAGER());

        //slither-disable-next-line arbitrary-send-eth

        (bool success, ) = payable(staderConfig.getUserWithdrawManager()).call{value: _amount}('');

        if (!success) {

            revert TransferFailed();

        }

        emit TransferredETHToUserWithdrawManager(_amount);

    }



Found in 2023-06-stader-labs/contracts/Auction.sol (function withdrawUnselectedBid(uint256 lotId) external override nonReentrant {):

        if (withdrawalAmount == 0) revert InSufficientETH();



        lotItem.bids[msg.sender] -= withdrawalAmount;



        // send the funds

        (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');

        if (!success) revert ETHWithdrawFailed();



        emit BidWithdrawn(lotId, msg.sender, withdrawalAmount);

    }





Found in 2023-06-stader-labs/contracts/SocializingPool.sol (function handleRewards(RewardsData calldata _rewardsData) external override nonReentrant {):



        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{

            value: _rewardsData.userETHRewards

        }();



        (bool success, ) = payable(staderConfig.getStaderTreasury()).call{value: _rewardsData.protocolETHRewards}('');

        if (!success) {

            revert ETHTransferFailed(staderConfig.getStaderTreasury(), _rewardsData.protocolETHRewards);

        }



        emit OperatorRewardsUpdated(



Found in 2023-06-stader-labs/contracts/VaultProxy.sol (function initialise():

     * of validatorWithdrawalVault/nodeELRewardVault for already deployed vaults*/

    fallback(bytes calldata _input) external payable returns (bytes memory) {

        address vaultImplementation = isValidatorWithdrawalVault

            ? staderConfig.getValidatorWithdrawalVaultImplementation()

            : staderConfig.getNodeELRewardVaultImplementation();

        (bool success, bytes memory data) = vaultImplementation.delegatecall(_input);

        if (!success) {

            revert(string(data));

        }

        return data;

    }



Found in 2023-06-stader-labs/contracts/StaderInsuranceFund.sol (function withdrawFund(uint256 _amount) external override nonReentrant {):

        if (address(this).balance < _amount || _amount == 0) {

            revert InvalidAmountProvided();

        }



        //slither-disable-next-line arbitrary-send-eth

        (bool success, ) = payable(msg.sender).call{value: _amount}('');

        if (!success) {

            revert TransferFailed();

        }

        emit FundWithdrawn(_amount);

    }



Found in 2023-06-stader-labs/contracts/UserWithdrawalManager.sol (function sendValue(address payable _recipient, uint256 _amount) internal {):

        if (address(this).balance < _amount) {

            revert InSufficientBalance();

        }



        //slither-disable-next-line arbitrary-send-eth

        (bool success, ) = _recipient.call{value: _amount}('');

        if (!success) {

            revert ETHTransferFailed();

        }

    }

}



Found in 2023-06-stader-labs/contracts/library/UtilLib.sol (function sendValue(address _receiver, uint256 _amount) internal {):

            : (totalETHBalance * DECIMALS) / totalETHXSupply;

        return newExchangeRate;

    }



    function sendValue(address _receiver, uint256 _amount) internal {

        (bool success, ) = payable(_receiver).call{value: _amount}('');

        if (!success) {

            revert TransferFailed();

        }

    }

}


------------------------------------------------------------------------ 

### Mitigation 

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.










# VULN 17 

## [GAS] Empty blocks should be removed or emit something
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 23 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

    constructor() {}


Found in line 17 at 2023-06-stader-labs/contracts/VaultProxy.sol:

    constructor() {}


Found in line 14 at 2023-06-stader-labs/contracts/NodeELRewardVault.sol:

    constructor() {}

------------------------------------------------------------------------ 

### Mitigation 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).










# VULN 18 

## [GAS] require() or revert() statements that check input arguments should be at the top of the function (Also restructured some if’s)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 47 at 2023-06-stader-labs/contracts/VaultProxy.sol:

            revert(string(data));

------------------------------------------------------------------------ 

### Mitigation 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.
