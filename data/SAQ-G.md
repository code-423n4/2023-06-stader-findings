## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Public function not called by the contract should be declared external instead  | 3 | - |
| [G-02] | Use != 0 instead of > 0 for unsigned integer comparison | 4 | - |
| [G-03] | Can make the variable outside the loop to save gas | 14 | - |
| [G-04] | Should use arguments instead of state variable  | 11 | - |
| [G-05] | Use assembly to check for address(0) | 2 | - |
| [G-06] | Before some functions, we should check some variables for possible gas save | 3 | - |
| [G-07] | Use selfbalance() instead of address(this).balance | 9 | - |
| [G-08] | Empty blocks should be removed or emit something | 3 | - |
| [G-09] | Duplicated require()/if() checks should be refactored to a modifier or function  | 2 | - |
| [G-10] | Use constants instead of type(uintx).max | 1 | - |
| [G-11] | Use nested if and, avoid multiple check combinations | 8 | - |
| [G-12] | Use assembly to write address storage values | 23 | - |
| [G-13] | Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it) | 4 | - |
| [G-14] | abi.encode() is less efficient than abi.encodepacked() | 15 | - |
| [G-15] | Do not calculate  constant | 1 | - |
| [G-16] | >= costs less gas than > | 13 | - |
| [G-17] | Use assembly to write address storage values | 11 | - |
| [G-18] | Minimize external calls | More files | - |
| [G-19] | Sort Solidity operations using short-circuit mode | 14 | - |
| [G-20] | Use hardcode address instead address(this) | 4 | - |
| [G-21] | Use Mappings Instead of Arrays | 2 | - |
| [G-22] | Use assembly for math (add, sub, mul, div) | 2 | - |
| [G-23] | Gas savings can be achieved by changing the model for assigning value to the structure (260 gas) | 1 | - |



## Gas Optimizations  

## [G-1] Public function not called by the contract should be declared external instead 

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so. 

```solidity
file: /contracts/SocializingPool.sol

163    function verifyProof(
        uint256 _index,
        address _operator,
        uint256 _amountSD,
        uint256 _amountETH,
        bytes32[] calldata _merkleProof
169     ) public view returns (bool) {


206       function getRewardCycleDetails(uint256 _index) public view returns (uint256 _startBlock, uint256 _endBlock) {


```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L163-L169


```solidity
file: /contracts/ETHx.sol

29      function initialize(address _admin, address _staderConfig) public initializer {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L29


## [G-2] Use != 0 instead of > 0 for unsigned integer comparison

When we say "use != 0 instead of > 0 for unsigned integer comparison," we mean that when checking if an unsigned integer value is non-zero, it's more gas-efficient to use the != operator instead of the > operator. This is because the != operator is a bitwise operation and is therefore more efficient than the arithmetic comparison performed by the > operator.

Here's an example of how to use != 0 instead of > 0 for unsigned integer comparison:

contract MyContract {
    uint public value;
    
    function isNonZero() public view returns (bool) {
        return value != 0;
    }
}

Instances:
```solidity
file:/contracts/PermissionedNodeRegistry.sol

299   if (totalDefectedKeys > 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L299


```solidity
file: /contracts/PermissionlessNodeRegistry.sol

209   if (frontRunValidatorsLength > 0) {        

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L209


```solidity
file: /contracts/SocializingPool.sol

119   if (totalAmountETH > 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L119


```solidity
file: /contracts/SocializingPool.sol

127   if (totalAmountSD > 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L127


## [G-3]  Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /contracts/PermissionedNodeRegistry.sol

/// @audit the  " uint256 validatorId " should declare outside the loop.

/// @audit     +      uint256 validatorId;
///           for (uint256 i; i < frontRunValidatorsLength; ) {
/// @audit      +          validatorId = validatorIdByPubkey[_frontRunPubkey[i]];  

272       for (uint256 i; i < frontRunValidatorsLength; ) {
273            uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];



/// @audit the  " uint256 validatorId " should declare outside the loop.

285       for (uint256 i; i < invalidSignatureValidatorsLength; ) {
286            uint256 validatorId = validatorIdByPubkey[_invalidSignaturePubkey[i]];



/// @audit the  " uint256 validatorId " should declare outside the loop.

317       for (uint256 i; i < withdrawnValidatorCount; ) {
318            uint256 validatorId = validatorIdByPubkey[_pubkeys[i]];



/// @audit the  " uint256 validatorId " should declare outside the loop.

534       for (uint256 i = _startIndex; i < _endIndex; ) {
535            uint256 validatorId = validatorIdsByOperatorId[operatorId][i];



/// @audit the  " uint256 validatorId " should declare outside the loop.

637       for (uint256 i = startIndex; i < endIndex; i++) {
638            uint256 validatorId = validatorIdsByOperatorId[operatorId][i];

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L272-L273


```solidity
file: /contracts/PermissionedPool.sol

/// @audit the  "  uint256 validatorToDeposit " should declare outside the loop.

/// @audit  +     uint256 validatorToDeposit;
///               for (uint256 i = 1; i < selectedOperatorCapacityLength; ) {
/// @audit  +      validatorToDeposit = selectedOperatorCapacity[i];

100        for (uint256 i = 1; i < selectedOperatorCapacityLength; ) {
101              uint256 validatorToDeposit = selectedOperatorCapacity[i];

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L100-L101


```solidity
file: /contracts/PermissionlessNodeRegistry.sol

/// @audit the  " uint256 validatorId " should declare outside the loop.

199      for (uint256 i; i < readyToDepositValidatorsLength; ) {
200            uint256 validatorId = validatorIdByPubkey[_readyToDepositPubkey[i]];



215      for (uint256 i; i < frontRunValidatorsLength; ) {
216            uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];



225      for (uint256 i; i < invalidSignatureValidatorsLength; ) {
226            uint256 validatorId = validatorIdByPubkey[_invalidSignaturePubkey[i]];



247      for (uint256 i; i < withdrawnValidatorCount; ) {
248            uint256 validatorId = validatorIdByPubkey[_pubkeys[i]];



415      for (uint256 i = _startIndex; i < _endIndex; ) {
416            uint256 validatorId = validatorIdsByOperatorId[operatorId][i];


543      for (uint256 i = startIndex; i < endIndex; i++) {
544            uint256 validatorId = validatorIdsByOperatorId[operatorId][i];

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L199-L200


```solidity
file: /contracts/PoolSelector.sol
/// @audit the  uint256 poolCapacity , uint256 poolDepositSize and  uint256 remainingValidatorsToDeposit should  declare 
///     outside the loop.

/// @audit   +   uint256 poolCapacity;
/// @audit   +   uint256 poolDepositSize;
/// @audit   +   uint256 remainingValidatorsToDeposit;
///              for (uint256 j; j < poolCount; ) {
/// @audit   +            poolCapacity = poolUtils.getQueuedValidatorCountByPool(poolIdArray[i]);
///  @audit   +            poolDepositSize = ETH_PER_NODE - poolUtils.getCollateralETH(poolIdArray[i]);
///  @audit   +            remainingValidatorsToDeposit = ethToDeposit / poolDepositSize; 

90     for (uint256 j; j < poolCount; ) {
91            uint256 poolCapacity = poolUtils.getQueuedValidatorCountByPool(poolIdArray[i]);
92            uint256 poolDepositSize = ETH_PER_NODE - poolUtils.getCollateralETH(poolIdArray[i]);
93            uint256 remainingValidatorsToDeposit = ethToDeposit / poolDepositSize;


```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L90-L93


```solidity
file: /contracts/StaderOracle.sol

/// @audit the  "  bytes32 pubkeyRoot " should declare outside the loop.

485      for (uint256 i; i < keyCount; ) {
486                bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_mapd.sortedPubkeys[i]);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L485


## [G-4] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas 

```solidity
file: /contracts/Auction.sol
   
///@audit the ' bidIncrement ' state variable should not use in emit.

45      emit BidIncrementUpdated(bidIncrement);

58      emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L45


```solidity
file: /contracts/PermissionedNodeRegistry.sol

/// @audit the ' maxNonTerminalKeyPerOperator ' is state variable.

419       emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L419

```solidity 
file: /contracts/PermissionlessNodeRegistry.sol

/// @audit the ' totalActiveValidatorCount ' is state variable.

384      emit IncreasedTotalActiveValidatorCount(totalActiveValidatorCount);

709      emit DecreasedTotalActiveValidatorCount(totalActiveValidatorCount);


/// @audit the ' nextQueuedValidatorIndex ' is state variable.

271      emit UpdatedNextQueuedValidatorIndex(nextQueuedValidatorIndex);


/// @audit the ' inputKeyCountLimit ' is state variable.

328      emit UpdatedInputKeyCountLimit(inputKeyCountLimit);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L384


```solidity
file: /contracts/SocializingPool.sol

/// @audit the ' totalOperatorETHRewardsRemaining ' is state variable.


96      emit OperatorRewardsUpdated(
97            _rewardsData.operatorETHRewards,
98             totalOperatorETHRewardsRemaining,
99            _rewardsData.operatorSDRewards,
100            totalOperatorSDRewardsRemaining
101        );

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L96-L101


```solidity
file: /contracts/StaderOracle.sol

/// @audit the ' erInspectionMode ' is state variable.

673       emit ERInspectionModeActivated(erInspectionMode, block.timestamp);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L673


```solidity
file: /contracts/VaultProxy.sol

/// @audit the ' owner ' is state variable.

71        emit UpdatedOwner(owner);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L71


```solidity
file: /contracts/factory/VaultFactory.sol

96       emit UpdatedVaultProxyImplementation(vaultProxyImplementation);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L96

## [G-5] Use assembly to check for address(0)

  Save 6 gas per instance.

```solidity
file: /contracts/UserWithdrawalManager.sol

96        if (_owner == address(0)) revert ZeroAddressReceived();

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L96


```solidity
file: /contracts/library/UtilLib.sol

22      if (_address == address(0)) revert ZeroAddress();

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L22



## [G-6] Before some functions, we should check some variables for possible gas save

### Detials 

 Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

```solidity
file: /contracts/Auction.sol
/// @audit  before transfer of amount should check the amount like bellow.
/// @audit  require(_sdAmount > 0 ," The amount must be greater than 0 ");

55      if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L55

```solidity
file: /contracts/SDCollateral.sol

/// @audit  before transfer of amount should check the amount for 0 like bellow.
/// @audit  require(_sdAmount > 0 ," The amount must be greater than 0 ");

47     if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L47


```solidity
file: /contracts/awalManager.solUserWithdr#L104

/// @audit  before transfer of amount should check the amount for 0 like bellow.
/// @audit  require(_ethXAmount > 0 ," The amount must be greater than 0 ");

104      IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/awalManager.solUserWithdr#L104

## [G-7]  Use selfbalance() instead of address(this).balance

```solidity
file: /contracts/NodeELRewardVault.sol

28          uint256 totalRewards = address(this).balance;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L28

```solidity
file: /contracts/PermissionedPool.sol

174       if (preDepositValidatorCount != 0 || address(this).balance == 0) {

178        value: address(this).balance

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L174

```solidity
file: /contracts/SocializingPool.sol

69          address(this).balance - totalOperatorETHRewardsRemaining

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L69

```solidity
file: /contracts/StaderInsuranceFund.sol

43          if (address(this).balance < _amount || _amount == 0) {


62           if (address(this).balance < _amount) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L43

```solidity
file: /contracts/UserWithdrawalManager.sol

224          if (address(this).balance < _amount) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L224


```solidity
file: /contracts/ValidatorWithdrawalVault.sol

34          uint256 totalRewards = address(this).balance;

99          uint256 contractBalance = address(this).balance;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L34


## [G-8] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.


```solidity
file: /contracts/NodeELRewardVault.sol

14     constructor() {}

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L14

```solidity
file: /ValidatorWithdrawalVault.sol
23        constructor() {}

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L23


```solidity
file: /contracts/VaultProxy.sol
 
17   constructor() {}

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L17

## [G-9] Duplicated require()/if() checks should be refactored to a modifier or function  
   
   to reduce code duplication and improve readability. 

•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /contracts/Auction.sol

/// @audit the duplicated if conditions.

82     if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

97     if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

108    if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

122    if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L82

```solidity
file: /contracts/library/UtilLib.sol

59          if (_addr != withdrawVaultAddress) {

75         if (_addr != withdrawVaultAddress) {

90         if (_addr != withdrawVaultAddress) {        

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L59

## [G-10] Use constants instead of type(uintx).max

type(uint120).max or type(uint112).max or type(uint256) etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /contracts/SDCollateral.sol

/// @audit the ' type(uint256).max ' should change to constant data type.
106       IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L106


## [G-11]  Use nested if and, avoid multiple check combinations

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.


```solidity
file: /contracts/SocializingPool.sol

46          if (_amountSD[i] == 0 && _amountETH[i] == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L146


```solidity
file: /contracts/StaderConfig.sol

513     if (
514           !(variablesMap[MIN_DEPOSIT_AMOUNT] != 0 &&
515                variablesMap[MIN_WITHDRAW_AMOUNT] != 0 &&
516                variablesMap[MIN_DEPOSIT_AMOUNT] <= variablesMap[MAX_DEPOSIT_AMOUNT] &&
517                variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MAX_WITHDRAW_AMOUNT] &&
518                variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MIN_DEPOSIT_AMOUNT] &&
519                variablesMap[MAX_WITHDRAW_AMOUNT] >= variablesMap[MAX_DEPOSIT_AMOUNT])
220        ) {


```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L513-L220


```solidity
file:/contracts/StaderOracle.sol

432       submissionCount == trustedNodesCount / 2 + 1 &&


664     if (
665            !(newExchangeRate >= (currentExchangeRate * (ER_CHANGE_MAX_BPS - erChangeLimit)) / ER_CHANGE_MAX_BPS &&
666                newExchangeRate <= ((currentExchangeRate * (ER_CHANGE_MAX_BPS + erChangeLimit)) / ER_CHANGE_MAX_BPS))
667        ) {


147     if (
148            submissionCount == trustedNodesCount / 2 + 1 &&
149            _exchangeRate.reportingBlockNumber > exchangeRate.reportingBlockNumber
150        ) {

186     if (
187            !staderConfig.onlyManagerRole(msg.sender) &&
188            erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY > block.number
189        ) {       


371    if (
372            submissionCount == trustedNodesCount / 2 + 1 &&
373            _validatorStats.reportingBlockNumber > validatorStats.reportingBlockNumber
374        ) {


```
https://github.com/code-423n4/2023-06-stader/blob/mainc/contracts/StaderOracle.sol#L432

```solidity
file: /contracts/ValidatorWithdrawalVault.sol

35       if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L35

## [G-12] Use assembly to write address storage values


```solidity
file: /contracts/Auction.sol

29      function initialize(address _admin, address _staderConfig) external initializer {

138     function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L29


```solidity
file: /contracts/ETHx.sol

/// @audit ' address '

29      function initialize(address _admin, address _staderConfig) public initializer {

47      function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) whenNotPaused {

56      function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {

77     function _beforeTokenTransfer(
78        address from,
79        address to,
80        uint256 amount
81    ) internal override whenNotPaused {                


85     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L29


```solidity
file: /contracts/OperatorRewardsCollector.sol

27      function initialize(address _admin, address _staderConfig) external initializer {

40     function depositFor(address _receiver) external payable {        

56     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L27


```solidity
file: /contracts/Penalty.sol

31   function initialize(
32        address _admin,
33        address _staderConfig,
34        address _ratedOracleAddress
35    ) external initializer {


83      function updateRatedOracleAddress(address _ratedOracleAddress) external override {

91     function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {        

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L31-L32


```solidity
file: /contracts/PermissionedNodeRegistry.sol

66       function initialize(address _admin, address _staderConfig) public initializer {

88       function whitelistPermissionedNOs(address[] calldata _permissionedNOs) external override {

106      function onboardNodeOperator(string calldata _operatorName, address payable _operatorRewardAddress)   

401      function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {


522      function getOperatorTotalNonTerminalKeys(
523        address _nodeOperator,
524        uint256 _startIndex,
525        uint256 _endIndex
526      ) public view override returns (uint64) {


```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L66


```solidity
file: /contracts/PermissionedPool.sol

40       function initialize(address _admin, address _staderConfig) public initializer {

248      function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L40 

```solidity
file: /contracts/PermissionlessNodeRegistry.sol

66       function initialize(address _admin, address _staderConfig) public initializer {

354      function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L66

```solidity
file: /contracts/PoolUtils.sol

25       function initialize(address _admin, address _staderConfig) public initializer {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L25


## [G-13] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity
file: /contracts/OperatorRewardsCollector.sol

46     function claim() external whenNotPaused {
47        address operator = msg.sender;
48        uint256 amount = balances[operator];
49        balances[operator] -= amount;  

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L47-L49


```solidity
file: /contracts/SDCollateral.sol

///@audit msg.sender  should not cache to the operator

43      function depositSDAsCollateral(uint256 _sdAmount) external override {
44        address operator = msg.sender;
45        operatorSDBalance[operator] += _sdAmount;

///@audit msg.sender  should not cache to the operator

58     function withdraw(uint256 _requestedSD) external override {
59        address operator = msg.sender;
60        uint256 opSDBalance = operatorSDBalance[operator];



```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L43-L45

```solidity
file: /contracts/SocializingPool.sol

///@audit msg.sender  should not cache to the operator

113          address operator = msg.sender;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L113

## [G-14] abi.encode() is less efficient than    abi.encodepacked()

for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /contracts/StaderOracle.sol

127         abi.encode(
128                msg.sender,
129                _exchangeRate.reportingBlockNumber,
130                _exchangeRate.totalETHBalance,
131                _exchangeRate.totalETHXSupply
132            )


135   abi.encode(_exchangeRate.reportingBlockNumber, _exchangeRate.totalETHBalance, _exchangeRate.totalETHXSupply)


221       abi.encode(
222                msg.sender,
223                _rewardsData.index,
224                _rewardsData.merkleRoot,
225                _rewardsData.poolId,
226                _rewardsData.operatorETHRewards,
227                _rewardsData.userETHRewards,
228                _rewardsData.protocolETHRewards,
229                _rewardsData.operatorSDRewards
230            )


233      abi.encode(
234                _rewardsData.index,
235                _rewardsData.merkleRoot,
236                _rewardsData.poolId,
237                _rewardsData.operatorETHRewards,
238                _rewardsData.userETHRewards,
239                _rewardsData.protocolETHRewards,
240                _rewardsData.operatorSDRewards
241            )

282          bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _sdPriceData.reportingBlockNumber));

283          bytes32 submissionCountKey = keccak256(abi.encode(_sdPriceData.reportingBlockNumber));

407          bytes memory encodedPubkeys = abi.encode(_withdrawnValidators.sortedPubkeys);

418          abi.encode(_withdrawnValidators.reportingBlockNumber, _withdrawnValidators.nodeRegistry, encodedPubkeys)

466          bytes memory encodedPubkeys = abi.encode(_mapd.sortedPubkeys);

469          bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _mapd.index, encodedPubkeys));

470          bytes32 submissionCountKey = keccak256(abi.encode(_mapd.index, encodedPubkeys));

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L127-L132

```solidity
file: /contracts/factory/VaultFactory.sol

40          bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));

54          bytes32 salt = sha256(abi.encode(_poolId, _operatorId));

67          bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));

77          bytes32 salt = sha256(abi.encode(_poolId, _operatorId));

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L40

## [G-15] Do not calculate  constant.

```solidity
file: /contracts/StaderOracle.sol

26       uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L26

## [G-16] >= costs less gas than >
 
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas


```solidity
file: /contracts/Auction.sol

109       if (lotItem.highestBidAmount > 0) revert LotWasAuctioned();

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L109

```solidity
file: /contracts/PermissionedNodeRegistry.sol

204          bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);

222          if (_numValidators > totalValidatorToDeposit) {

299          if (totalDefectedKeys > 0) {        

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L204

```solidity
file: /contracts/PermissionlessNodeRegistry.sol

209          if (frontRunValidatorsLength > 0) {

305          if (address(feeRecipientAddress).balance > 0) {        

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L209

```solidity
file: /contracts/SocializingPool.sol

119          if (totalAmountETH > 0) {

127          if (totalAmountSD > 0) {        

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L119


```solidity
file: /contracts/StaderOracle.sol

121          if (_exchangeRate.reportingBlockNumber % updateFrequencyMap[ETHX_ER_UF] > 0) {

274          if (_sdPriceData.reportingBlockNumber % updateFrequencyMap[SD_PRICE_UF] > 0) {

328          if (_validatorStats.reportingBlockNumber % updateFrequencyMap[VALIDATOR_STATS_UF] > 0) {

403          if (_withdrawnValidators.reportingBlockNumber % updateFrequencyMap[WITHDRAWN_VALIDATORS_UF] > 0){                    

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L121


```solidity
file: /contracts/ValidatorWithdrawalVault.sol

115          if (totalRewards > 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L115

## [G-17] Use assembly to write address storage values


By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```solidity
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```
Instances:
```solidity
File: /contracts/Auction.sol

37    staderConfig = IStaderConfig(_staderConfig);

140    staderConfig = IStaderConfig(_staderConfig);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L37


```solidity
File: /contracts/ETHx.sol

37    staderConfig = IStaderConfig(_staderConfig);

87    staderConfig = IStaderConfig(_staderConfig);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L37

```solidity
File: /contracts/OperatorRewardsCollector.sol

34    staderConfig = IStaderConfig(_staderConfig);

41    balances[_receiver] += msg.value;

58    staderConfig = IStaderConfig(_staderConfig);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L34


```solidity
File: /contracts/Penalty.sol

42    staderConfig = IStaderConfig(_staderConfig);

43    ratedOracleAddress = _ratedOracleAddress;

86    ratedOracleAddress = _ratedOracleAddress;

93    staderConfig = IStaderConfig(_staderConfig);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L42


## [G-18] Minimize external calls

 External calls to other contracts are expensive in terms of gas. Try to minimize external calls by using in-line code or caching data in memory.

```solidity
file: more files

Write in more contract files.

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts


## [G-19] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows
 f(x) || g(y) 
 f(x) && g(y)
 

```solidity
file: /contracts/PermissionedNodeRegistry.sol

693          if (_pubkeyLength != _preDepositSignatureLength || _pubkeyLength != _depositSignatureLength) {

697          if (keyCount == 0 || keyCount > inputKeyCountLimit) {

734          return (validator.status == ValidatorStatus.PRE_DEPOSIT || validator.status == ValidatorStatus.DEPOSITED);

740     return
741            !(validator.status == ValidatorStatus.WITHDRAWN ||
742                validator.status == ValidatorStatus.FRONT_RUN ||
743                validator.status == ValidatorStatus.INVALID_SIGNATURE);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L693

```solidity
file: /contracts/PermissionedPool.sol

174          if (preDepositValidatorCount != 0 || address(this).balance == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L174

```solidity
file: /contracts/PermissionlessNodeRegistry.sol

649          if (_pubkeyLength != _preDepositSignatureLength || _pubkeyLength != _depositSignatureLength) {


693   return
694            !(validator.status == ValidatorStatus.WITHDRAWN ||
695                validator.status == ValidatorStatus.FRONT_RUN ||
696                validator.status == ValidatorStatus.INVALID_SIGNATURE);        


713          if (_validatorId == 0 || validatorRegistry[_validatorId].status != ValidatorStatus.INITIALIZED) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L649

```solidity
file: /contracts/PoolSelector.sol

103           if (ethToDeposit < ETH_PER_NODE || selectedValidatorCount >= poolAllocationMaxSize) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L103

```solidity
file: /contracts/PoolUtils.sol

282      if (
283            INodeRegistry(IStaderPoolBase(_poolAddress).getNodeRegistry()).POOL_ID() != _poolId ||
284            isExistingPoolId(_poolId)
285        ) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L282

```solidity
file: /contracts/SDCollateral.sol

127          if ((_minThreshold > _withdrawThreshold) || (_minThreshold > _maxThreshold)) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L127

```solidity
file: /contracts/SocializingPool.sol

170         if (_index == 0 || _index > lastReportedRewardsData.index) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L170

```solidity
file: /contracts/StaderOracle.sol

532          if (_erChangeLimit == 0 || _erChangeLimit > ER_CHANGE_MAX_BPS) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L532

```solidity
file: /contracts/UserWithdrawalManager.sol

98          if (assets < staderConfig.getMinWithdrawAmount() || assets > staderConfig.getMaxWithdrawAmount()) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L98


## [G-20] Use hardcode address instead address(this)

When we say "use hardcode address instead of address(this)," we mean that if the contract's address is needed in the code, it's more gas-efficient to hardcode the address as a constant rather than using the address(this) expression. This is because using address(this) requires additional gas consumption to compute the contract's address at runtime.

Here's an example :
```solidity 
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```
Instances:
```solidity
file: /contracts/NodeELRewardVault.sol

25        uint8 poolId = IVaultProxy(address(this)).poolId();
26        uint256 operatorId = IVaultProxy(address(this)).id();
27        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
28        uint256 totalRewards = address(this).balance;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L25-L28

```solidity
file: /contracts/PermissionedPool.sol

174    if (preDepositValidatorCount != 0 || address(this).balance == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L174

```solidity
file: /contracts/PermissionedPool.sol

178    value: address(this).balance

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L178


```solidity
file: /contracts/ValidatorWithdrawalVault.sol

31        uint8 poolId = VaultProxy(payable(address(this))).poolId();
32        uint256 validatorId = VaultProxy(payable(address(this))).id();
33        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
34        uint256 totalRewards = address(this).balance;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L31-L34


## [G-21] Use Mappings Instead of Arrays


When we say "use mappings instead of arrays," we mean that when storing and accessing data in a smart contract, it's more gas-efficient to use mappings instead of arrays. This is because mappings do not require iteration to find a specific element, whereas arrays do.


```solidity
file: /contracts/Penalty.sol

125   uint256[] memory violatedEpochs = IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L125


```solidity
file: /contracts/PermissionedNodeRegistry.sol

202    uint256[] memory remainingOperatorCapacity = new uint256[](nextOperatorId);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L202


## [G-22] Use assembly for math (add, sub, mul, div)

when performing basic math operations in a smart contract, it's more gas-efficient to use assembly code to perform the operation instead of Solidity's built-in math functions. This is because Solidity's built-in math functions require additional gas consumption to perform the operation.



```solidity
file: /contracts/PermissionedNodeRegistry.sol

629  uint256 endIndex = startIndex + _pageSize;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L629


```solidity
file: /contracts/PermissionlessNodeRegistry.sol

500   uint256 endIndex = startIndex + _pageSize;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L500


## [G-23] Gas savings can be achieved by changing the model for assigning value to the structure (260 gas) 


When we say "change the model for assigning value to a structure," we mean that when initializing or assigning values to a structure, it's more gas-efficient to use a single assignment statement that includes all of the values for the structure, instead of assigning each value one by one. This is because each individual assignment requires additional gas consumption, so reducing the number of assignments can lead to significant gas savings.

```solidity
file: /contracts/SDCollateral.sol

131         poolThresholdbyPoolId[_poolId] = PoolThresholdInfo({
132            minThreshold: _minThreshold,
133            maxThreshold: _maxThreshold,
134            withdrawThreshold: _withdrawThreshold,
135            units: _units
136        });

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L131-L136
