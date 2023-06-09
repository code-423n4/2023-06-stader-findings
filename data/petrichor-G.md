# Gas Optimization

# Summary 

|      |  Issue   |  Instance  |
|------|----------|------------|
|[G-01]|Make The Variable Outside The Loop To Save Gas |12|
|[G-02]| Use assembly to write address storage values|21|
|[G-03]|Use nested if statements instead of &&|2|
|[G-04]|Sort Solidity operations using short-circuit mode|13|
|[G-05]| public functions to external|21|
|[G-06]| Amounts should be checked for 0 before calling a transfer|4|
|[G-07]|Do not calculate constants|1|
|[G-08]|Do not shrink Variables|6|
|[G-09]| abi.encode() is less efficient than abi.encodePacked()|17|
|[G-10]|Change public state variable visibility to private|1|
|[G-11]|Use != 0 instead of > 0 for unsigned integer comparison|4|
|[G-12]|Use hardcode address instead address(this)|4|
|[G-13]| Use Assembly To Check For address(0)|1|
|[G-14]|Use constants instead of type(uintx).max|1|
|[G-15]|Duplicated require()/if() checks should be refactored to a modifier or function|3|
|[G-16]|Use assembly to hash instead of Solidity|14|
|[G-17]|Loop best practice to save gas|12|
|[G-18]|Remove the initializer modifier|8|
|[G-19]|use Mappings Instead of Arrays|2|
|[G-20]|Use assembly for math (add, sub, mul, div)|2|
|[G-21]|Use assembly when getting a contract’s balance of ETH|4|
|[G-22]|Replace state variable reads and writes within loops with local variable reads and writes.|2||
|[G-23]|Gas savings can be achieved by changing the model for assigning value to the structure (260 gas) |1|
|[G‑24]| Avoid contract existence checks by using low level calls





## [G-01] Can Make The Variable Outside The Loop To Save Gas 
In Solidity, variables that are declared inside a loop will be created and stored in memory each time the loop executes. This can lead to inefficient gas usage, as the cost of executing a loop can increase significantly if the loop body creates new variables each time it executes.
On the other hand, if you declare the variable outside the loop, the variable will be created once and reused across multiple iterations of the loop. This can be more efficient because it reduces the amount of gas needed to create and store the variable.

```solidity

104   bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L104

```solidity
106   uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L106

```solidity
107   uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L107


```solidity
92    address operator = _permissionedNOs[i];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L92


```solidity
157   address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L157

```solidity
273   uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L273


```solidity
286   uint256 validatorId = validatorIdByPubkey[_invalidSignaturePubkey[i]];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L286

```soldity
318   uint256 validatorId = validatorIdByPubkey[_pubkeys[i]];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L318

```solidity
535   uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L535

```solidity
638   uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L638


```solidity
101  uint256 validatorToDeposit = selectedOperatorCapacity[i];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L101

```solidity
110  uint256 index = nextQueuedValidatorIndex;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L110


## [G-02] Use assembly to write address storage values

By using assembly to directly write the address to storage, we can save gas compared to using Solidity's built-in storage operations. This is because the assembly code is more efficient and optimized for writing directly to storage.

```solidity
37    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L37

```solidity
140    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L140



```solidity
37    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L37

```solidity
87    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L87

```solidity
34    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L34

```solidity
41    balances[_receiver] += msg.value;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L41

```solidity
58    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L58


```solidity
42    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L42

```solidity
43    ratedOracleAddress = _ratedOracleAddress;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L43

```solidity
86    ratedOracleAddress = _ratedOracleAddress;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L86

```solidity
93    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L93


```solidity
72   staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L72

```solidity
407  operatorStructById[operatorId].operatorRewardAddress = _rewardAddress;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L407

```solidity
461   staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L461


```solidity
45   staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L45

```solidity
149    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L149


```solidtity
29   staderConfig = IStaderConfig(_staderConfig); 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L29
 
```solidity
 33    staderConfig = IStaderConfig(_staderConfig);
```
 https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L33
 
 ```solidity
 47     staderConfig = IStaderConfig(_staderConfig);
```
 https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L47

```solidity
 60   staderConfig = IStaderConfig(_staderConfig);
 ```
  https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L60

```solidity
 86   staderConfig = IStaderConfig(_staderConfig);
```
 https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L86



## [G-03]Use nested if statements instead of &&

Instead of using the && operator, you can use nested if statements to achieve the same result. Using a logical AND (&&) in an if statement can consume more gas compared to using nested if statements.


```solidity
146   if (_amountSD[i] == 0 && _amountETH[i] == 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L146

```solidity
35   if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L35


## [G-04] Sort Solidity operations using short-circuit mode

Short-circuiting is a technique used in Solidity contract development that utilizes logical operators (OR/AND) to control the order of execution of different operations with varying gas costs. It involves placing the operations with lower gas costs at the beginning and those with higher gas costs at the end. This way, if the low-cost operation at the beginning is successful, the subsequent high-cost Ethereum virtual machine operation can be skipped, resulting in significant gas savings. Essentially, short-circuiting allows the program to take a shortcut and bypass more expensive operations when it's possible to do so.

example:
function transfer(address recipient, uint amount) public {
    require(recipient != address(0) || amount > 0, "Invalid transfer parameters");
    // Perform transfer logic
}



```solidity
693   if (_pubkeyLength != _preDepositSignatureLength || _pubkeyLength != _depositSignatureLength) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L693

```solidity
697   if (keyCount == 0 || keyCount > inputKeyCountLimit) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L697

```solidity
174   if (preDepositValidatorCount != 0 || address(this).balance == 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L174

```solidity
649   if (_pubkeyLength != _preDepositSignatureLength || _pubkeyLength != _depositSignatureLength) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L649

```solidity
653   if (keyCount == 0 || keyCount > inputKeyCountLimit) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L653

```solidity
713   if (_validatorId == 0 || validatorRegistry[_validatorId].status != ValidatorStatus.INITIALIZED) {            
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L713

```solidity
103     if (ethToDeposit < ETH_PER_NODE || selectedValidatorCount >= poolAllocationMaxSize) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L103

```solidity
127   if ((_minThreshold > _withdrawThreshold) || (_minThreshold > _maxThreshold)) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L127

```solidity
170   if (_index == 0 || _index > lastReportedRewardsData.index) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L170

```solidity
44   if (address(this).balance < _amount || _amount == 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L44

```solidity
532   if (_erChangeLimit == 0 || _erChangeLimit > ER_CHANGE_MAX_BPS) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L532

```solidity
171   if (assets > maxDeposit() || assets < minDeposit()) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L171

```solidity
98    if (assets < staderConfig.getMinWithdrawAmount() || assets > staderConfig.getMaxWithdrawAmount()) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L98



## [G-05] public functions to external.

When we say that "external call cost is less expensive than that of public functions," we mean that calling an external function is generally cheaper in terms of gas costs than calling a public function. This is because external functions can only be accessed externally, and therefore, they don't require the additional checks and validation that public functions do. The Solidity compiler is able to optimize external functions more effectively, resulting in lower gas costs.

example:

contract MyContract {
    uint public value;
    
    function setValue(uint newValue) public {
        require(newValue > value, "New value must be greater than current value");
        value = newValue;
    }
    
    function getValue() external view returns (uint) {
        return value;
    }
}



```solidity
29   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L29


```solidity
66  function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L66


```solidity
40   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L40


```solidity
66   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L66

```solidity
440   function getTotalQueuedValidatorCount() public view override returns (uint256) {            
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L440


```solidity
38   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L38


```solidtity
25    function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L25

```solidity
26   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L26    
```solidity
205   function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L205

```solidity
39    function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L39

```solidity
62   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L62

```solidity
571   function getERReportableBlock() public view override returns (uint256) {
        return getReportableBlockFor(ETHX_ER_UF);
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L571

```solidity
582   function getSDPriceReportableBlock() public view override returns (uint256) {
        return getReportableBlockFor(SD_PRICE_UF);
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L582

```solidity
586   function getValidatorStatsReportableBlock() public view override returns (uint256) {
        return getReportableBlockFor(VALIDATOR_STATS_UF);
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L586

```solidity
590    function getWithdrawnValidatorReportableBlock() public view override returns (uint256) {
        return getReportableBlockFor(WITHDRAWN_VALIDATORS_UF);
    }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L590


```solidity
50   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#50

```solidity
140   function convertToShares(uint256 _assets) public view override returns (uint256) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#140

```solidity
145   function convertToAssets(uint256 _shares) public view override returns (uint256) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#145

```solidity
54   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L54


```solidity
34     function deployWithdrawVault(
        uint8 _poolId,
        uint256 _operatorId,
        uint256 _validatorCount,
        uint256 _validatorId
    ) public override onlyRole(NODE_REGISTRY_CONTRACT) returns (address) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L34

```solidity
62    function computeWithdrawVaultAddress(
        uint8 _poolId,
        uint256 _operatorId,
        uint256 _validatorCount
    ) public view override returns (address) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L62




## [G-06] Amounts should be checked for 0 before calling a transfer

When transferring tokens or ether in a Solidity smart contract using the transfer function, it's important to check the amount being transferred to avoid wasting gas by transferring zero amounts. This is because transferring zero amounts is still a valid operation, but it has no effect, and it can consume gas unnecessarily.

To avoid this waste of gas, it's recommended to check the amount being transferred before calling the transfer function. This way, we can avoid making an unnecessary transfer and save gas in the process.

example:

function sendTokens(address recipient, uint amount) public {
    require(amount > 0, "Amount must be greater than zero");
    require(token.balanceOf(msg.sender) >= amount, "Insufficient balance");
    token.transfer(recipient, amount);
}





```solidity
55    if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L55

```solidity
87    if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {    
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L87

```solidity
82    super._beforeTokenTransfer(from, to, amount);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L82

```solidity
68    if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L68



 ## [G-07] Do not calculate constants
When we say "do not calculate constants," we mean that if a value is constant and known at compile-time, it should be hard-coded into the code instead of being calculated at runtime. This avoids the need for expensive calculations during contract execution and can help reduce gas costs.

Here's an example of how to avoid calculating constants:
contract MyContract {
    uint constant public MAX_VALUE = 100;
    uint public value;
    
    function setValue(uint newValue) public {
        require(newValue <= MAX_VALUE, "Value exceeds maximum");
        value = newValue;
    }
}


```solidity
26    uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L26


## [G-08]Do not shrink Variables

When we say "do not shrink variables," we mean that if a variable needs to hold a certain range of values, it should be declared with the appropriate data type to avoid unnecessary conversions and potential data loss. Shrinking a variable's data type to the smallest possible size can result in additional gas costs due to the need for conversions and can even lead to data loss if the value exceeds the variable's range.

Here's an example of how to avoid shrinking variables:

contract MyContract {
    uint public value;
    
    function setValue(uint256 newValue) public {
        require(newValue <= 100, "Value exceeds maximum");
        value = newValue;
    }
}





```solidity
30    uint8 public constant override POOL_ID = 1;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L30

```solidity
31    uint16 public override inputKeyCountLimit;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L31

```solidity
32    uint64 public override maxNonTerminalKeyPerOperator;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L32


```solidity
31    uint8 public constant override POOL_ID = 2;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L31

```solidity
32    uint16 public override inputKeyCountLimit;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L32

```solidity
33    uint64 public override maxNonTerminalKeyPerOperator;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L33



## [G-09] abi.encode() is less efficient than abi.encodePacked()

When we say that abi.encodePacked() is less gas-intensive than abi.encode(), we mean that abi.encodePacked() is generally more efficient in terms of gas usage because it does not add any additional overhead or padding to the encoded data. In contrast, abi.encode() adds additional padding to the encoded data, resulting in higher gas costs.

Here's an example of how to use abi.encodePacked() to save gas:

contract MyContract {
    function encodeData(uint256 value, address recipient, bytes32 data) public returns (bytes memory) {
        return abi.encodePacked(value, recipient, data);
    }
}




```solidity
127    abi.encode(
                msg.sender,
                _exchangeRate.reportingBlockNumber,
                _exchangeRate.totalETHBalance,
                _exchangeRate.totalETHXSupply
            )
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L127

```solidity
135    abi.encode(_exchangeRate.reportingBlockNumber, _exchangeRate.totalETHBalance, _exchangeRate.totalETHXSupply)
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L135

```solidity
221    abi.encode(
                msg.sender,
                _rewardsData.index,
                _rewardsData.merkleRoot,
                _rewardsData.poolId,
                _rewardsData.operatorETHRewards,
                _rewardsData.userETHRewards,
                _rewardsData.protocolETHRewards,
                _rewardsData.operatorSDRewards
            )
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L221

```solidity    
233     abi.encode(
                _rewardsData.index,
                _rewardsData.merkleRoot,
                _rewardsData.poolId,
                _rewardsData.operatorETHRewards,
                _rewardsData.userETHRewards,
                _rewardsData.protocolETHRewards,
                _rewardsData.operatorSDRewards
            )
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L233

```solidity
282     bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _sdPriceData.reportingBlockNumber));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L282

```solidity
283    bytes32 submissionCountKey = keccak256(abi.encode(_sdPriceData.reportingBlockNumber));   
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L283


```solidity
334     abi.encode(
                msg.sender,
                _validatorStats.reportingBlockNumber,
                _validatorStats.exitingValidatorsBalance,
                _validatorStats.exitedValidatorsBalance,
                _validatorStats.slashedValidatorsBalance,
                _validatorStats.exitingValidatorsCount,
                _validatorStats.exitedValidatorsCount,
                _validatorStats.slashedValidatorsCount
            )
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L334

```solidity
346      abi.encode(
                _validatorStats.reportingBlockNumber,
                _validatorStats.exitingValidatorsBalance,
                _validatorStats.exitedValidatorsBalance,
                _validatorStats.slashedValidatorsBalance,
                _validatorStats.exitingValidatorsCount,
                _validatorStats.exitedValidatorsCount,
                _validatorStats.slashedValidatorsCount
            )  

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L346

```solidity
407      bytes memory encodedPubkeys = abi.encode(_withdrawnValidators.sortedPubkeys);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L407

```solidity
410       abi.encode(
                msg.sender,
                _withdrawnValidators.reportingBlockNumber,
                _withdrawnValidators.nodeRegistry,
                encodedPubkeys
            )
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L410

```solidity
418       abi.encode(_withdrawnValidators.reportingBlockNumber, _withdrawnValidators.nodeRegistry, encodedPubkeys)
``` 
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L418

```solidity
469       bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _mapd.index, encodedPubkeys));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L469

```solidity
470       bytes32 submissionCountKey = keccak256(abi.encode(_mapd.index, encodedPubkeys));

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L470


```solidity
40     bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L40

```solidity
54     bytes32 salt = sha256(abi.encode(_poolId, _operatorId));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L54

```solidity
67     bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L67

```solidity
77     bytes32 salt = sha256(abi.encode(_poolId, _operatorId));
        
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L77



 ## [G-10] Change public state variable visibility to private

```solidity
14   address public override owner;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L14



## [G-11]Use != 0 instead of > 0 for unsigned integer comparison

When we say "use != 0 instead of > 0 for unsigned integer comparison," we mean that when checking if an unsigned integer value is non-zero, it's more gas-efficient to use the != operator instead of the > operator. This is because the != operator is a bitwise operation and is therefore more efficient than the arithmetic comparison performed by the > operator.

Here's an example of how to use != 0 instead of > 0 for unsigned integer comparison:

contract MyContract {
    uint public value;
    
    function isNonZero() public view returns (bool) {
        return value != 0;
    }
}


```solidity
299   if (totalDefectedKeys > 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L299

```solidity
209   if (frontRunValidatorsLength > 0) {         
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L209


```solidity
119   if (totalAmountETH > 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L119

```solidity
127   if (totalAmountSD > 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L127


## [G-12] Use hardcode address instead address(this)

When we say "use hardcode address instead of address(this)," we mean that if the contract's address is needed in the code, it's more gas-efficient to hardcode the address as a constant rather than using the address(this) expression. This is because using address(this) requires additional gas consumption to compute the contract's address at runtime.

Here's an example :
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}




```solidity
        uint8 poolId = IVaultProxy(address(this)).poolId();
        uint256 operatorId = IVaultProxy(address(this)).id();
        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
        uint256 totalRewards = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L25-L28

```solidity
174    if (preDepositValidatorCount != 0 || address(this).balance == 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L174

```solidity


178    value: address(this).balance
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L178


```solidity
       31 uint8 poolId = VaultProxy(payable(address(this))).poolId();
        uint256 validatorId = VaultProxy(payable(address(this))).id();
        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
      34  uint256 totalRewards = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L31-L34

## [G-13] Use Assembly To Check For address(0)

```solidity
96    if (_owner == address(0)) revert ZeroAddressReceived();
```
 https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L96


## [G-14] Use constants instead of type(uintx).max

When we say "use constants instead of type(uintX).max," we mean that when initializing a variable with the maximum value of an unsigned integer type, it's more gas-efficient to use a constant variable rather than the type(uintX).max expression. This is because the type(uintX).max expression requires additional gas consumption to compute the maximum value at runtime.


```solidity
106   IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L106


## [G-15] Duplicated require()/if() checks should be refactored to a modifier or function


```solidity
59    if (_addr != withdrawVaultAddress) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L59

```solidity
75    if (_addr != withdrawVaultAddress) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L75

```solidity

90    if (_addr != withdrawVaultAddress) {        
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L90




 

 ## [G-16]   Use assembly to hash instead of Solidity

When we say "use assembly to hash instead of Solidity," we mean that when hashing data in a smart contract, it's more gas-efficient to use assembly code to perform the hash operation instead of Solidity's built-in hash functions. This is because Solidity's built-in hash functions require additional gas consumption to perform the operation.

Here's an example:

contract MyContract {
    function sha256(bytes memory data) public view returns (bytes32) {
        bytes32 hash;
        assembly {
            hash := sha256(add(data, 32), mload(data))
        }
        return hash;
    }
}


```solidity
174    bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L174

```solidity
126    bytes32 nodeSubmissionKey = keccak256(
            abi.encode(
                msg.sender,
                _exchangeRate.reportingBlockNumber,
                _exchangeRate.totalETHBalance,
                _exchangeRate.totalETHXSupply
            )
        );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L126


```solidity
134    bytes32 submissionCountKey = keccak256(
            abi.encode(_exchangeRate.reportingBlockNumber, _exchangeRate.totalETHBalance, _exchangeRate.totalETHXSupply)
        );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L134

```solidity
220    bytes32 nodeSubmissionKey = keccak256(
            abi.encode(
                msg.sender,
                _rewardsData.index,
                _rewardsData.merkleRoot,
                _rewardsData.poolId,
                _rewardsData.operatorETHRewards,
                _rewardsData.userETHRewards,
                _rewardsData.protocolETHRewards,
                _rewardsData.operatorSDRewards
            )
        );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L220

```solidity        
232     bytes32 submissionCountKey = keccak256(
            abi.encode(
                _rewardsData.index,
                _rewardsData.merkleRoot,
                _rewardsData.poolId,
                _rewardsData.operatorETHRewards,
                _rewardsData.userETHRewards,
                _rewardsData.protocolETHRewards,
                _rewardsData.operatorSDRewards
            )
        );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L232


```solidity
282    bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _sdPriceData.reportingBlockNumber));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L282

```solidity
283    bytes32 submissionCountKey = keccak256(abi.encode(_sdPriceData.reportingBlockNumber));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L283


```solidity
333   bytes32 nodeSubmissionKey = keccak256(
            abi.encode(
                msg.sender,
                _validatorStats.reportingBlockNumber,
                _validatorStats.exitingValidatorsBalance,
                _validatorStats.exitedValidatorsBalance,
                _validatorStats.slashedValidatorsBalance,
                _validatorStats.exitingValidatorsCount,
                _validatorStats.exitedValidatorsCount,
                _validatorStats.slashedValidatorsCount
            )
        );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L333


```solidity
345    bytes32 submissionCountKey = keccak256(
            abi.encode(
                _validatorStats.reportingBlockNumber,
                _validatorStats.exitingValidatorsBalance,
                _validatorStats.exitedValidatorsBalance,
                _validatorStats.slashedValidatorsBalance,
                _validatorStats.exitingValidatorsCount,
                _validatorStats.exitedValidatorsCount,
                _validatorStats.slashedValidatorsCount
            )
        );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L345


```solidity
409      bytes32 nodeSubmissionKey = keccak256(
            abi.encode(
                msg.sender,
                _withdrawnValidators.reportingBlockNumber,
                _withdrawnValidators.nodeRegistry,
                encodedPubkeys
            )
        );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L409

```solidity
417     bytes32 submissionCountKey = keccak256(
            abi.encode(_withdrawnValidators.reportingBlockNumber, _withdrawnValidators.nodeRegistry, encodedPubkeys)
        );     
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L417


```solidity
469     bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _mapd.index, encodedPubkeys));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L469


```solidity
470     bytes32 submissionCountKey = keccak256(abi.encode(_mapd.index, encodedPubkeys));           
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L470


```solidity
16       bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L16




## [G-17]Loop best practice to save gas.




```solidity
177  unchecked {
                ++i;
     }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L177

```solidity
214  unchecked {
                ++i;
     }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L214


```solidity
279  unchecked {
                ++i;
     }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L279

```solidity
292  unchecked {
                ++i;
     }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L292

```solidity
326  unchecked {
                ++i;
     }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L326

```solidity
492  unchecked {
                ++i;
     }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L492

```solidity
539  unchecked {
                ++i;
     }                            
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L539


```solidity
163  unchecked {
                ++i;
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L163

```solidity
204  unchecked {
                ++i;
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L204


```solidity
220 unchecked {
                ++i;
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L220

```solidity
230 unchecked {
                ++i;
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L230

```solidity
256  unchecked {
                ++i;
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L256

```solidity
420  unchecked {
                ++i;
    }                    
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L420


## [G-18]|Remove the initializer modifier.
When we say "remove the initializer modifier," we mean that instead of defining a separate function with the initializer modifier to initialize a contract's state variables, we can simply initialize them directly in the constructor function. This can be more gas-efficient because it avoids the additional gas costs associated with calling a separate initialization function.

```solidity
29   function initialize(address _admin, address _staderConfig) external initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L29

```solidity
29   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L29

```solidity
27   function initialize(address _admin, address _staderConfig) external initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L27

```solidity
31    function initialize(
        address _admin,
        address _staderConfig,
        address _ratedOracleAddress
    ) external initializer {        
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L31

```solidity
66   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L66

```solidity
40    function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L40

```solidity
66   function initialize(address _admin, address _staderConfig) public initializer {           
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L66

```solidity
38    function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L38







## [G-19]use Mappings Instead of Arrays.


When we say "use mappings instead of arrays," we mean that when storing and accessing data in a smart contract, it's more gas-efficient to use mappings instead of arrays. This is because mappings do not require iteration to find a specific element, whereas arrays do.


```solidity
125   uint256[] memory violatedEpochs = IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L125

```solidity
202    uint256[] memory remainingOperatorCapacity = new uint256[](nextOperatorId);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L202



## [G-20]|Use assembly for math (add, sub, mul, div)

When we say "use assembly for math operations," we mean that when performing basic math operations in a smart contract, it's more gas-efficient to use assembly code to perform the operation instead of Solidity's built-in math functions. This is because Solidity's built-in math functions require additional gas consumption to perform the operation.



```solidity
629  uint256 endIndex = startIndex + _pageSize;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L629

```solidity
500   uint256 endIndex = startIndex + _pageSize;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L500


## [G-21] Use assembly when getting a contract’s balance of ETH

When we say "use assembly to get a contract's balance of Ether," we mean that when retrieving the balance of Ether held by a contract, it's more gas-efficient to use assembly code to perform the operation instead of Solidity's built-in address.balance property. This is because Solidity's built-in address.balance property requires additional gas consumption to perform the operation.

```solidity
28   uint256 totalRewards = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L28

```solidity
178   value: address(this).balance
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L178

```solidity
34    uint256 totalRewards = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L34

```solidity
99    uint256 contractBalance = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L99




## [G-22]Replace state variable reads and writes within loops with local variable reads and writes.


When we say "replace state variable reads and writes within loops with local variable reads and writes," we mean that when accessing state variables within loops, it's more gas-efficient to read the state variable once and store it in a local variable, and subsequently, read and write to the local variable within the loop instead of repeatedly accessing the state variable. This is because reading and writing to state variables is more gas-intensive than reading and writing to local variables.
```solidity
206    for (uint256 i = 1; i < nextOperatorId; ) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L206


```solidity
488    for (uint256 i = 1; i < nextOperatorId; ) {    
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L488


## [G-23] Gas savings can be achieved by changing the model for assigning value to the structure (260 gas) 


When we say "change the model for assigning value to a structure," we mean that when initializing or assigning values to a structure, it's more gas-efficient to use a single assignment statement that includes all of the values for the structure, instead of assigning each value one by one. This is because each individual assignment requires additional gas consumption, so reducing the number of assignments can lead to significant gas savings.

```solidity
131      poolThresholdbyPoolId[_poolId] = PoolThresholdInfo({
            minThreshold: _minThreshold,
            maxThreshold: _maxThreshold,
            withdrawThreshold: _withdrawThreshold,
            units: _units
        });
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L131-L136






## [G‑27] Avoid contract existence checks by using low level calls
 when invoking a contract function, it's more gas-efficient to use a low-level call instead of Solidity's built-in contract existence checks. This is because Solidity's built-in contract existence checks require additional gas consumption to perform the check, which can be costly when invoked repeatedly.




```solidity
95     uint256[] memory selectedOperatorCapacity = IPermissionedNodeRegistry(nodeRegistryAddress)
            .allocateValidatorsAndUpdateOperatorId(requiredValidators);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L95

```solidity
114    uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdsByOperatorId(i, index);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L114

```solidity
139    uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdByPubkey(_pubkey[i]);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L139

```solidity
140    (, , , bytes memory depositSignature, address withdrawVaultAddress, , , ) = INodeRegistry(
                nodeRegistryAddress
            ).validatorRegistry(validatorId);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L140


```solidity
143     bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(
                withdrawVaultAddress
            );            
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L143

```solidity
186    return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getTotalQueuedValidatorCount();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L186

```solidity
183     return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getTotalActiveValidatorCount();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L183

```solidity
205     INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getOperatorTotalNonTerminalKeys(
                _nodeOperator,
                _startIndex,
                _endIndex
            );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L205

```solidity
218     return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getCollateralETH();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L218

```solidity
227     return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).isExistingPubkey(_pubkey);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L227

```solidity
232     return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).isExistingOperator(_operAddr);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L232

```solidity
298      bytes memory withdrawCredential = IVaultFactory(_vaultFactory).getValidatorWithdrawCredential(
            withdrawVaultAddress
        );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L298

```solidity
95    address withdrawVault = IVaultFactory(vaultFactory).computeWithdrawVaultAddress(
                INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID(),
                _operatorId,
                _operatorTotalKeys + i
            );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L95
```solidity
100   bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(withdrawVault);            
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L100

```solidity
136   uint256 depositQueueStartIndex = IPermissionlessNodeRegistry(nodeRegistryAddress).nextQueuedValidatorIndex();

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L136

```solidity
138   uint256 validatorId = IPermissionlessNodeRegistry(nodeRegistryAddress).queuedValidators(i);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L138
```solidity
165   return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getTotalQueuedValidatorCount();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L165
```solidity
172    return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getTotalActiveValidatorCount();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L172
```solidity
184     INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getOperatorTotalNonTerminalKeys(
                _nodeOperator,
                _startIndex,
                _endIndex
            );
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L184


```solidity
192      return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getCollateralETH();

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L192

```solidity
201   return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).isExistingPubkey(_pubkey);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L201

```solidity
206     return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).isExistingOperator(_operAddr);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L206

```solidity
248     (, bytes memory pubkey, , bytes memory depositSignature, address withdrawVaultAddress, , , ) = INodeRegistry(
            _nodeRegistryAddress
        ).validatorRegistry(_validatorId);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L248

```solidity
252    bytes memory withdrawCredential = IVaultFactory(_vaultFactoryAddress).getValidatorWithdrawCredential(
            withdrawVaultAddress
        );        
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L252

```solidity
File: /contracts/ValidatorWithdrawalVault.sol
145    (, bytes memory pubkey, , , , , , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);

149    return IPenalty(_staderConfig.getPenaltyContract()).totalPenaltyAmount(pubkey);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L145

```solidity
197     uint256 selectedPoolCapacity = IPoolSelector(staderConfig.getPoolSelector()).computePoolAllocationForDeposit(
            _poolId,
            (availableETHForNewDeposit / poolDepositSize)
        );
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L197

```solidity        
227   (uint256[] memory selectedPoolCapacity, uint8[] memory poolIdArray) = IPoolSelector(
            staderConfig.getPoolSelector()
        ).poolAllocationForExcessETHDeposit(availableETHForNewDeposit);               
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L227

```solidity
32   (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
            .calculateRewardShare(poolId, totalRewards);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L32

```solidity
256     address socializingPool = IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(
                _rewardsData.poolId
            );
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L256

```solidity
576      (, , uint256 currentEndBlock) = ISocializingPool(
            IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(_poolId)
        ).getRewardDetails();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L576

```solidity
608    ISocializingPool(IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(_poolId))
                .getCurrentRewardsIndex();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L608

```solidity
125   uint256[] memory violatedEpochs = IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L125

```solidity
92   return IStaderPoolBase(poolAddressById[_poolId]).protocolFee();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L92

```solidity
143  return IStaderPoolBase(poolAddressById[_poolId]).getSocializingPoolAddress();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L143

```solidity
163  return IStaderPoolBase(poolAddressById[_poolId]).getNodeRegistry();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L163

