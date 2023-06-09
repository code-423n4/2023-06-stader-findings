# Gas Optimization

# Summary 

|      |  Issue   | Instance |
|------|----------|----------|
|[G-01]|Avoid contract existence checks by using low level calls|42|
|[G-02]|Use != 0 instead of > 0 for unsigned integer comparison|4|
|[G-03]|Use hardcode address instead address(this)|10|
|[G-04]|Use nested if statements instead of &&|2|
|[G-05]|Duplicated require()/if() checks should be refactored to a modifier or function|7|
|[G-06]|Change public state variable visibility to private|1|
|[G-07]|Use assembly to hash instead of Solidity|14|
|[G-08]|Unnecessary computation|2|
|[G-09]|Can Make The Variable Outside The Loop To Save Gas |12|
|[G-10]|Use Assembly To Check For address(0)|1|
|[G-11]|Loop best practice to save gas|13|
|[G-12]|Avoid using state variable in emit|10|
|[G-13]|Remove the initializer modifier|8|
|[G-14]|use Mappings Instead of Arrays|2|
|[G-15]|Use assembly for math (add, sub, mul, div)|2|
|[G-16]|Use assembly when getting a contract’s balance of ETH|4|
|[G-17]|Do not shrink Variables|6|
|[G-18]|When possible, use assembly instead of unchecked{++i}|21|
|[G-19]|Use constants instead of type(uintx).max|1|
|[G-20]|Gas savings can be achieved by changing the model for assigning value to the structure (260 gas) |1|
|[G-21]| Do not calculate constants |1|
|[G-22]|Amounts should be checked for 0 before calling a transfer|4|
|[G-23]|public functions to external|10|
|[G-24]|Use assembly to write address storage values|21|
|[G-25]|Sort Solidity operations using short-circuit mode|13|

# Detials
## [G‑01] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File: /contracts/UserWithdrawalManager.sol
97    uint256 assets = IStaderStakePoolManager(staderConfig.getStakePoolManager()).previewWithdraw(_ethXAmount);

126   uint256 exchangeRate = IStaderStakePoolManager(poolManager).getExchangeRate();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L97

```solidity
File: /contracts/PermissionedNodeRegistry.sol
157    address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(
                POOL_ID,
                operatorId,
                operatorTotalKeys + i, //operator totalKeys
                nextValidatorId
       );

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L157

```solidity
File: /contracts/PermissionedPool.sol
95     uint256[] memory selectedOperatorCapacity = IPermissionedNodeRegistry(nodeRegistryAddress)
            .allocateValidatorsAndUpdateOperatorId(requiredValidators);

114    uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdsByOperatorId(i, index);

139    uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdByPubkey(_pubkey[i]);

140    (, , , bytes memory depositSignature, address withdrawVaultAddress, , , ) = INodeRegistry(
                nodeRegistryAddress
            ).validatorRegistry(validatorId);

143     bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(
                withdrawVaultAddress
            );            

186    return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getTotalQueuedValidatorCount();

183     return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getTotalActiveValidatorCount();

205     INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getOperatorTotalNonTerminalKeys(
                _nodeOperator,
                _startIndex,
                _endIndex
            );
218     return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getCollateralETH();

227     return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).isExistingPubkey(_pubkey);

232     return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).isExistingOperator(_operAddr);

298      bytes memory withdrawCredential = IVaultFactory(_vaultFactory).getValidatorWithdrawCredential(
            withdrawVaultAddress
        );


```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L95

```solidity
File: /contracts/PermissionlessPool.sol
95    address withdrawVault = IVaultFactory(vaultFactory).computeWithdrawVaultAddress(
                INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID(),
                _operatorId,
                _operatorTotalKeys + i
            );

100   bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(withdrawVault);            

136   uint256 depositQueueStartIndex = IPermissionlessNodeRegistry(nodeRegistryAddress).nextQueuedValidatorIndex();

138   uint256 validatorId = IPermissionlessNodeRegistry(nodeRegistryAddress).queuedValidators(i);

165   return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getTotalQueuedValidatorCount();

172    return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getTotalActiveValidatorCount();

184     INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getOperatorTotalNonTerminalKeys(
                _nodeOperator,
                _startIndex,
                _endIndex
            );

192      return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getCollateralETH();

201   return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).isExistingPubkey(_pubkey);

206     return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).isExistingOperator(_operAddr);

248     (, bytes memory pubkey, , bytes memory depositSignature, address withdrawVaultAddress, , , ) = INodeRegistry(
            _nodeRegistryAddress
        ).validatorRegistry(_validatorId);

252    bytes memory withdrawCredential = IVaultFactory(_vaultFactoryAddress).getValidatorWithdrawCredential(
            withdrawVaultAddress
        );        
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L95

```solidity
File: /contracts/ValidatorWithdrawalVault.sol
145    (, bytes memory pubkey, , , , , , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);

149    return IPenalty(_staderConfig.getPenaltyContract()).totalPenaltyAmount(pubkey);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L145

```solidity
File: /contracts/StaderStakePoolsManager.sol
197     uint256 selectedPoolCapacity = IPoolSelector(staderConfig.getPoolSelector()).computePoolAllocationForDeposit(
            _poolId,
            (availableETHForNewDeposit / poolDepositSize)
        );
227   (uint256[] memory selectedPoolCapacity, uint8[] memory poolIdArray) = IPoolSelector(
            staderConfig.getPoolSelector()
        ).poolAllocationForExcessETHDeposit(availableETHForNewDeposit);               
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L197

```solidity
32   (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
            .calculateRewardShare(poolId, totalRewards);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L32

```solidity
File: /contracts/StaderOracle.sol
256     address socializingPool = IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(
                _rewardsData.poolId
            );
576      (, , uint256 currentEndBlock) = ISocializingPool(
            IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(_poolId)
        ).getRewardDetails();

608    ISocializingPool(IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(_poolId))
                .getCurrentRewardsIndex();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L256

```solidity
File: /contracts/Penalty.sol
125   uint256[] memory violatedEpochs = IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L125

```solidity
File: /contracts/PoolUtils.sol
92   return IStaderPoolBase(poolAddressById[_poolId]).protocolFee();

143  return IStaderPoolBase(poolAddressById[_poolId]).getSocializingPoolAddress();

163  return IStaderPoolBase(poolAddressById[_poolId]).getNodeRegistry();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L92

## [G-02]Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```
function someFunction(uint256 x) public returns (bool) {
    // Use != 0 for unsigned integer comparison
    if (x != 0) {
        // Do something
        return true;
    }
    // Do something else
    return false;
}
```
and this is bad code

```
function someFunction(uint256 x) public returns (bool) {
    // Use > 0 for unsigned integer comparison
    if (x > 0) {
        // Do something
        return true;
    }
    // Do something else
    return false;
}
```

```solidity
File: /contracts/PermissionedNodeRegistry.sol
299   if (totalDefectedKeys > 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L299

### Recommend code
```
+   if (totalDefectedKeys != 0) {
            decreaseTotalActiveValidatorCount(totalDefectedKeys);
            IPermissionedPool(permissionedPool).transferETHOfDefectiveKeysToSSPM(totalDefectedKeys);
    }       
```

```solidity
File: /contracts/PermissionlessNodeRegistry.sol
209   if (frontRunValidatorsLength > 0) {         
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L209

### Recommend code
```
+   if (frontRunValidatorsLength != 0) {
            IStaderInsuranceFund(staderConfig.getStaderInsuranceFund()).depositFund{
                value: frontRunValidatorsLength * FRONT_RUN_PENALTY
            }();
    }      
```

```solidity
File: /contracts/SocializingPool.sol
119   if (totalAmountETH > 0) {

127   if (totalAmountSD > 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L119

### Recommend code
```
+    if (totalAmountETH != 0) {
            totalOperatorETHRewardsRemaining -= totalAmountETH;
            (success, ) = payable(operatorRewardsAddr).call{value: totalAmountETH}('');
            if (!success) {
                revert ETHTransferFailed(operatorRewardsAddr, totalAmountETH);
            }
     }

+     if (totalAmountSD != 0) {
            totalOperatorSDRewardsRemaining -= totalAmountSD;
            if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {
                revert SDTransferFailed();
            }
       }  
``` 
## [G-03] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)


```solidity
        uint8 poolId = IVaultProxy(address(this)).poolId();
        uint256 operatorId = IVaultProxy(address(this)).id();
        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
        uint256 totalRewards = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L25-L28

### Recommend code
```
function withdraw() external override {
    // Pre-calculate and use a hardcoded contract address
    address contractAddress = 0x1234567890123456789012345678901234567890;

+   uint8 poolId = IVaultProxy(contractAddress).poolId();
+   uint256 operatorId = IVaultProxy(contractAddress).id();
+   IStaderConfig staderConfig = IVaultProxy(contractAddress).staderConfig();
+   uint256 totalRewards = contractAddress.balance;
    if (totalRewards == 0) {
        revert NotEnoughRewardToWithdraw();
    }
    (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
        .calculateRewardShare(poolId, totalRewards);

    // Distribute rewards
    IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{value: userShare}();
    // slither-disable-next-line arbitrary-send-eth
    UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
    address operator = UtilLib.getOperatorAddressByOperatorId(poolId, operatorId, staderConfig);
    IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
        operator
    );

    emit Withdrawal(protocolShare, operatorShare, userShare);
}
```


```solidity
File: /contracts/PermissionedPool.sol
174    if (preDepositValidatorCount != 0 || address(this).balance == 0) {

178    value: address(this).balance
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L174

### Recommend code
```
function transferExcessETHToSSPM() external nonReentrant {
    // Pre-calculate and use a hardcoded contract address
    address contractAddress = 0x1234567890123456789012345678901234567890;

+   if (preDepositValidatorCount != 0 || contractAddress.balance == 0) {
        revert CouldNotDetermineExcessETH();
    }
    IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExcessEthFromPool{
+       value: contractAddress.balance
    }(INodeRegistry((staderConfig).getPermissionedNodeRegistry()).POOL_ID());
}

```


```solidity
File: /contracts/ValidatorWithdrawalVault.sol
        uint8 poolId = VaultProxy(payable(address(this))).poolId();
        uint256 validatorId = VaultProxy(payable(address(this))).id();
        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
        uint256 totalRewards = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L31-L34

### Recommend code
```
function distributeRewards() external override {
    // Pre-calculate and use a hardcoded contract address
    address contractAddress = 0x1234567890123456789012345678901234567890;

+   uint8 poolId = VaultProxy(payable(contractAddress)).poolId();
+   uint256 validatorId = VaultProxy(payable(contractAddress)).id();
+   IStaderConfig staderConfig = VaultProxy(payable(contractAddress)).staderConfig();
+   uint256 totalRewards = contractAddress.balance;
    if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
        emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
        revert InvalidRewardAmount();
    }
    if (totalRewards == 0) {
        revert NotEnoughRewardToDistribute();
    }
    (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
        .calculateRewardShare(poolId, totalRewards);

    // Distribute rewards
    IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}();
    UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
    IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
        getOperatorAddress(poolId, validatorId, staderConfig)
    );
    emit DistributedRewards(userShare, operatorShare, protocolShare);
}

```
## [G-04]Use nested if statements instead of &&
 Using a logical AND (&&) in an if statement can consume more gas compared to using nested if statements. The reason behind this is that the Solidity compiler generates additional bytecode to evaluate the combined condition in a single step. This extra bytecode incurs a slightly higher gas cost. On the other hand, with nested if statements, the conditions are evaluated sequentially, and if any condition fails, the subsequent conditions are not evaluated, resulting in slightly lower gas consumption.

Like This
```
contract NestedIfTest {

    //Execution cost: 22334 gas
    function funcBad(uint256 input) public pure returns (string memory) { 
       if (input<10 && input>0 && input!=6){ 
           return "If condition passed";
       } 

   }

    //Execution cost: 22294 gas
    function funcGood(uint256 input) public pure returns (string memory) { 
    if (input<10) { 
        if (input>0){
            if (input!=6){
                return "If condition passed";
            }
        }
    }
   }
}
   
```



```solidity
File: /contracts/SocializingPool.sol
146   if (_amountSD[i] == 0 && _amountETH[i] == 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L146

### Recommend code
```
if (_amountSD[i] == 0) {
    if (_amountETH[i] == 0) {
        revert InvalidAmount();
    }
}
```

```solidity
File: /contracts/ValidatorWithdrawalVault.sol
35   if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L35

### Recommend code
```
if (!staderConfig.onlyOperatorRole(msg.sender)) {
    if (totalRewards > staderConfig.getRewardsThreshold()) {
        emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
        revert InvalidRewardAmount();
    }
}
```
## [G-05] Duplicated require()/if() checks should be refactored to a modifier or function
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

Recommendation
You can consider adding a modifier like below
```
 modifer check (address checkToAddress) {    
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");  
      _; 
 }
```

```solidity
File: /contracts/library/UtilLib.sol
59    if (_addr != withdrawVaultAddress) {

75    if (_addr != withdrawVaultAddress) {

90    if (_addr != withdrawVaultAddress) {        
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L59

```solidity
File: /contracts/Auction.sol

82     if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

97     if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

108    if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

122    if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L82

## [G-06] Change public state variable visibility to private
it's generally a good practice to limit the visibility of state variables to the minimum necessary level. This means that you should use the private visibility modifier for state variables whenever possible, and avoid using the public modifier unless it is absolutely necessary.

1.Security: Public state variables can be read and modified by anyone on the blockchain, which can make your contract vulnerable to attacks. By using the private modifier, you can limit access to your state variables and reduce the risk of malicious actors exploiting them.

2.Encapsulation: Using private state variables can help to encapsulate the internal workings of your contract and make it easier to reason about and maintain. By only exposing the necessary interfaces to the outside world, you can reduce the complexity and potential for errors in your contract.

3.Gas costs: Public state variables can be more expensive to read and write than private state variables, since Solidity generates additional getter and setter functions for public variables. By using private state variables, you can reduce the gas cost of your contract and improve its efficiency.

```solidity
File: /contracts/VaultProxy.sol
14   address public override owner;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L14

## [G-07] Use assembly to hash instead of Solidity
using assembly to perform hashing operations can sometimes be more gas-efficient than using Solidity built-in hashing functions. This is because Solidity's hashing functions, such as keccak256, are implemented as Solidity functions, which require additional gas to execute.
Here's an example of how to perform a keccak256 hash using assembly:

```
function hashUsingAssembly(string memory _input) public pure returns (bytes32) {
    bytes memory inputBytes = bytes(_input);
    bytes32 output;

    assembly {
        output := keccak256(add(inputBytes, 32), mload(inputBytes))
    }

    return output;
}
```
In the above code, the hashUsingAssembly function takes a string input and returns the keccak256 hash of the input as a bytes32 value.

The assembly block contains the actual hashing operation. The add(inputBytes, 32) instruction adds the memory offset of the input bytes to the value 32, which skips the first 32 bytes of the input. This is because the first 32 bytes of the input contain the length of the input, which is not included in the hash. The mload(inputBytes) instruction loads the length of the input from memory, which is used as the length parameter for the keccak256 function.

Note that while using assembly can be more gas-efficient, it can also be more error-prone and difficult to read and maintain. Therefore, it should only be used when absolutely necessary and when the gas savings are significant enough to justify the added complexity.

```solidity
File: /contracts/SocializingPool.sol
174    bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L174

```solidity
File: /contracts/StaderOracle.sol
126    bytes32 nodeSubmissionKey = keccak256(
            abi.encode(
                msg.sender,
                _exchangeRate.reportingBlockNumber,
                _exchangeRate.totalETHBalance,
                _exchangeRate.totalETHXSupply
            )
        );

134    bytes32 submissionCountKey = keccak256(
            abi.encode(_exchangeRate.reportingBlockNumber, _exchangeRate.totalETHBalance, _exchangeRate.totalETHXSupply)
        );


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
        
282    bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _sdPriceData.reportingBlockNumber));

283    bytes32 submissionCountKey = keccak256(abi.encode(_sdPriceData.reportingBlockNumber));

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

409      bytes32 nodeSubmissionKey = keccak256(
            abi.encode(
                msg.sender,
                _withdrawnValidators.reportingBlockNumber,
                _withdrawnValidators.nodeRegistry,
                encodedPubkeys
            )
        );

417     bytes32 submissionCountKey = keccak256(
            abi.encode(_withdrawnValidators.reportingBlockNumber, _withdrawnValidators.nodeRegistry, encodedPubkeys)
        );     

469     bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _mapd.index, encodedPubkeys));
470     bytes32 submissionCountKey = keccak256(abi.encode(_mapd.index, encodedPubkeys));           
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#126


```solidity
File: /contracts/factory/VaultFactory.sol
16       bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L16

## [G-08] Unnecessary computation
When emitting an event that includes a new and an old value, it is cheaper in gas to avoid caching the old value in memory. Instead, emit the event, then save the new value in storage.

Proof of Concept
Instances include:

```
OwnableProxyDelegation.sol
function _setOwner
Recommended Mitigation
```
Replace
```
address oldOwner = _owner;
_owner = newOwner;
emit OwnershipTransferred(oldOwner, newOwner)
```
with
```
emit OwnershipTransferred(_owner_, newOwner)
_owner = newOwner;
```


```solidity
File: /contracts/UserWithdrawalManager.sol
80    emit UpdatedFinalizationBatchLimit(_finalizationBatchLimit);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L80

```solidity
File: /contracts/factory/VaultFactory.sol
96    emit UpdatedVaultProxyImplementation(vaultProxyImplementation);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L96


## [G-09] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.
By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

```solidity
File: /contracts/Penalty.sol
104   bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);

106   uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);

107   uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L104


```solidity
File: /contracts/PermissionedNodeRegistry.sol
92    address operator = _permissionedNOs[i];

157   address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(

273   uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];

286   uint256 validatorId = validatorIdByPubkey[_invalidSignaturePubkey[i]];

318   uint256 validatorId = validatorIdByPubkey[_pubkeys[i]];

535   uint256 validatorId = validatorIdsByOperatorId[operatorId][i];

638   uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L92


```solidity
File: /contracts/PermissionedPool.sol
101  uint256 validatorToDeposit = selectedOperatorCapacity[i];

110  uint256 index = nextQueuedValidatorIndex;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L101

## [G-10] Use Assembly To Check For address(0)
it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);
        
        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)
            
            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)
            
            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```
In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary

```solidity
File: /contracts/UserWithdrawalManager.sol
96    if (_owner == address(0)) revert ZeroAddressReceived();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L96

## [G-11] Loop best practice to save gas
```
function Plusi() public view {
		for(uint i=0; i<10;) {
			console.log("Number ==",i);
			unchecked{
				++i;
			}
		}
	}

best practice
-----------------------------------------------------
for (uint i = 0; i < length; i = unchecked_inc(i)) {
    // do something that doesn't change the value of i
}

function unchecked_inc(uint i) returns (uint) {
    unchecked {
        return i + 1;
    }
}
```

```solidity
File: /contracts/PermissionedNodeRegistry.sol
177  unchecked {
                ++i;
     }
214  unchecked {
                ++i;
     }  
279  unchecked {
                ++i;
     }
292  unchecked {
                ++i;
     }
326  unchecked {
                ++i;
     }
492  unchecked {
                ++i;
     }
539  unchecked {
                ++i;
     }                            
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L177
```solidity
File: /contracts/PermissionedNodeRegistry.sol
163  unchecked {
                ++i;
    }
204  unchecked {
                ++i;
    }
220 unchecked {
                ++i;
    }
230 unchecked {
                ++i;
    }
256  unchecked {
                ++i;
    }
420  unchecked {
                ++i;
    }                    
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L163


## [G-12]Avoid using state variable in emit

```solidity
File: /contracts/Auction.sol
44   emit AuctionDurationUpdated(duration);

45   emit BidIncrementUpdated(bidIncrement);

58   emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);

148  emit AuctionDurationUpdated(duration);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L44

```solidity
File: /contracts/PermissionedNodeRegistry.sol
175   emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId);

419   emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);

430   emit UpdatedInputKeyCountLimit(inputKeyCountLimit);

478    emit IncreasedTotalActiveValidatorCount(totalActiveValidatorCount);

667    emit OnboardedOperator(msg.sender, _operatorRewardAddress, nextOperatorId - 1);

749   emit DecreasedTotalActiveValidatorCount(totalActiveValidatorCount);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L175

## [G-13] Remove the initializer modifier
If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

```solidity
File: /contracts/Auction.sol
29   function initialize(address _admin, address _staderConfig) external initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L29

```solidity
File: /contracts/ETHx.sol
29   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L29

```solidity
File: /contracts/OperatorRewardsCollector.sol
27   function initialize(address _admin, address _staderConfig) external initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L27

```solidity
File: /contracts/Penalty.sol
31    function initialize(
        address _admin,
        address _staderConfig,
        address _ratedOracleAddress
    ) external initializer {        
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L31

```solidity
File: /contracts/PermissionedNodeRegistry.sol
66   function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L66

```solidity
File: /contracts/PermissionedPool.sol
40    function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L40

```solidity
File: /contracts/PermissionlessNodeRegistry.sol
66   function initialize(address _admin, address _staderConfig) public initializer {           
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L66

```solidity
File: /contracts/PermissionlessPool.sol
38    function initialize(address _admin, address _staderConfig) public initializer {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L38

## [G-14]use Mappings Instead of Arrays
There are two data types to describe lists of data in Solidity, arrays and maps, and their syntax and structure are quite different, allowing each to serve a distinct purpose. While arrays are packable and iterable, mappings are less expensive.

For example, creating an array of cars in Solidity might look like this:

string cars[];
cars = ["ford", "audi", "chevrolet"];

Let’s see how to create a mapping for cars:


mapping(uint => string) public cars

When using the mapping keyword, you will specify the data type for the key (uint) and the value (string). Then you can add some data using the constructor function.

 constructor() public {
        cars[101] = "Ford";
        cars[102] = "Audi";
        cars[103] = "Chevrolet";
    }
}

Except where iteration is required or data types can be packed, it is advised to use mappings to manage lists of data in order to conserve gas. This is beneficial for both memory and storage.

An integer index can be used as a key in a mapping to control an ordered list. Another advantage of mappings is that you can access any value without having to iterate through an array as would otherwise be necessary. 

```solidity
File: /contracts/Penalty.sol
125   uint256[] memory violatedEpochs = IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L125

```solidity
File: /contracts/PermissionedNodeRegistry.sol
202    uint256[] memory remainingOperatorCapacity = new uint256[](nextOperatorId);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L202

## [G-15] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.
```solidity
//addition in Solidity
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```
Gas: 303
```solidity
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263

```solidity
File: /contracts/PermissionedNodeRegistry.sol
629  uint256 endIndex = startIndex + _pageSize;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L629

```solidity
File: /contracts/PermissionlessNodeRegistry.sol
500   uint256 endIndex = startIndex + _pageSize;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L500

## [G-16] Use assembly when getting a contract’s balance of ETH
You can use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.
```
function addressInternalBalance() public returns (uint256) {
    return address(this).balance;
}
```
Gas: 148
```
function assemblyInternalBalance() public returns (uint256) {
    assembly {
        let c := selfbalance()
        mstore(0x00, c)
        return(0x00, 0x20)
    }
}
```
Gas: 133



```solidity
28   uint256 totalRewards = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L28

```solidity
File: /contracts/PermissionedPool.sol
178   value: address(this).balance
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L178

```solidity
File: /contracts/ValidatorWithdrawalVault.sol
34    uint256 totalRewards = address(this).balance;

99    uint256 contractBalance = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L34

##  [G-17]Do not shrink Variables
This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.

```solidity
File: /contracts/PermissionlessNodeRegistry.sol
30    uint8 public constant override POOL_ID = 1;

31    uint16 public override inputKeyCountLimit;

32    uint64 public override maxNonTerminalKeyPerOperator;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L30


```solidity
File: /contracts/PermissionedNodeRegistry.sol
31    uint8 public constant override POOL_ID = 2;

32    uint16 public override inputKeyCountLimit;

33    uint64 public override maxNonTerminalKeyPerOperator;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L31


## [G-18] When possible, use assembly instead of unchecked{++i}
You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however this limits the functionality you can achieve.

```solidity
//loop with unchecked{++i}
function uncheckedPlusPlusI() public pure {
    uint256 j = 0;
    for (uint256 i; i < 10; ) {
        j++;
        unchecked {
            ++i;
        }
    }
}
```
Gas: 1329
```solidity
//loop with inline assembly
function inlineAssemblyLoop() public pure {
    assembly {
        let j := 0
        for {
            let i := 0
        } lt(i, 10) {
            i := add(i, 0x01)
        } {
            j := add(j, 0x01)
        }
    }
}
```
Gas: 709

```solidity
File: /contracts/Penalty.sol
116    unchecked {
                ++i;
      }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L116

```solidity
File: /contracts/PermissionedNodeRegistry.sol
177  unchecked {
                ++i;
     }
214  unchecked {
                ++i;
     }  
279  unchecked {
                ++i;
     }
292  unchecked {
                ++i;
     }
326  unchecked {
                ++i;
     }
492  unchecked {
                ++i;
     }
539  unchecked {
                ++i;
     }                            
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L177

```solidity
File: /contracts/PermissionedPool.sol
121  unchecked {
                ++i;
     }
163  unchecked {
                ++i;
     }    
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L121

```solidity
File: /contracts/PermissionedNodeRegistry.sol
163  unchecked {
                ++i;
    }
204  unchecked {
                ++i;
    }
220 unchecked {
                ++i;
    }
230 unchecked {
                ++i;
    }
256  unchecked {
                ++i;
    }
420  unchecked {
                ++i;
    }                    
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L163

```solidity
File: /contracts/PermissionlessPool.sol
116  unchecked {
                ++i;
    }
146  unchecked {
                ++i;
    }    
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L116

```solidity
File : /contracts/PoolSelector.sol
107   unchecked {
                ++j;
     }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L107

```solidity
 File: /contracts/UserWithdrawalManager.sol
148    unchecked {
                ++requestId;
     }
216    unchecked {
                ++requestId;
     }     
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L148

 ## [G-19] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```solidity
File :/contracts/SDCollateral.sol
106   IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L106

## [G-20] Gas savings can be achieved by changing the model for assigning value to the structure (260 gas)    
     function deploy(address optionFactory, address identifier0, address identifier1) internal returns (address optionPair) {        
     parameter = Parameter({optionFactory: optionsFactory, token0: token0, token1: token1});
     
     +parameter.optionFactory = optionFactory;
     +parameter.token0 = token0;+              parameter.token1 = token1; 

```solidity
File :/contracts/SDCollateral.sol
131      poolThresholdbyPoolId[_poolId] = PoolThresholdInfo({
            minThreshold: _minThreshold,
            maxThreshold: _maxThreshold,
            withdrawThreshold: _withdrawThreshold,
            units: _units
        });
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L131-L136

## [G-21] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
File: /contracts/StaderOracle.sol
26    uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L26

## [G-22] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```solidity
File: /contracts/Auction.sol
55    if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {

87    if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {    
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L55

```solidity
File: /contracts/ETHx.sol
82    super._beforeTokenTransfer(from, to, amount);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L82

```solidity
File :/contracts/SDCollateral.sol
68    if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L68

## [G-23] public functions to external
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.


```solidity
File: /contracts/PermissionlessNodeRegistry.sol
440   function getTotalQueuedValidatorCount() public view override returns (uint256) {            
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L440

```solidity
205   function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L205


```solidity
File: /contracts/StaderOracle.sol
571   function getERReportableBlock() public view override returns (uint256) {
        return getReportableBlockFor(ETHX_ER_UF);
    }

582   function getSDPriceReportableBlock() public view override returns (uint256) {
        return getReportableBlockFor(SD_PRICE_UF);
    }

586   function getValidatorStatsReportableBlock() public view override returns (uint256) {
        return getReportableBlockFor(VALIDATOR_STATS_UF);
    }

590    function getWithdrawnValidatorReportableBlock() public view override returns (uint256) {
        return getReportableBlockFor(WITHDRAWN_VALIDATORS_UF);
    }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L571


```solidity
File: /contracts/StaderStakePoolsManager.sol
140   function convertToShares(uint256 _assets) public view override returns (uint256) {

145   function convertToAssets(uint256 _shares) public view override returns (uint256) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#140

```solidity
File: /contracts/factory/VaultFactory.sol
34     function deployWithdrawVault(
        uint8 _poolId,
        uint256 _operatorId,
        uint256 _validatorCount,
        uint256 _validatorId
    ) public override onlyRole(NODE_REGISTRY_CONTRACT) returns (address) {
62    function computeWithdrawVaultAddress(
        uint8 _poolId,
        uint256 _operatorId,
        uint256 _validatorCount
    ) public view override returns (address) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L34

## [G-24] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```
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


```solidity
File: /contracts/PermissionedNodeRegistry.sol
72   staderConfig = IStaderConfig(_staderConfig);

407  operatorStructById[operatorId].operatorRewardAddress = _rewardAddress;

461   staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L72


```solidity
File: /contracts/PermissionlessPool.sol
45   staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L45


```solidity
File : /contracts/PoolSelector.sol
149    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L149


```solodtity
File: /contracts/PoolUtils.sol
29   staderConfig = IStaderConfig(_staderConfig); 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L29
 
```solidity
 File :/contracts/SDCollateral.sol
 33    staderConfig = IStaderConfig(_staderConfig);

 47     staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L33

```solidity
 File: /contracts/UserWithdrawalManager.sol
 60   staderConfig = IStaderConfig(_staderConfig);

 86   staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L60

## [G-25] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.
```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
```solidity
File: /contracts/PermissionedNodeRegistry.sol

693   if (_pubkeyLength != _preDepositSignatureLength || _pubkeyLength != _depositSignatureLength) {

697   if (keyCount == 0 || keyCount > inputKeyCountLimit) {

174   if (preDepositValidatorCount != 0 || address(this).balance == 0) {


```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L693

```solidity
File: /contracts/PermissionlessNodeRegistry.sol
649   if (_pubkeyLength != _preDepositSignatureLength || _pubkeyLength != _depositSignatureLength) {

653   if (keyCount == 0 || keyCount > inputKeyCountLimit) {

713   if (_validatorId == 0 || validatorRegistry[_validatorId].status != ValidatorStatus.INITIALIZED) {            
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L649

```solidity
File: /contracts/PoolSelector.sol
103     if (ethToDeposit < ETH_PER_NODE || selectedValidatorCount >= poolAllocationMaxSize) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L103

```solidity
File: /contracts/SDCollateral.sol
127   if ((_minThreshold > _withdrawThreshold) || (_minThreshold > _maxThreshold)) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L127

```solidity
File: /contracts/SocializingPool.sol
170   if (_index == 0 || _index > lastReportedRewardsData.index) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L170

```solidity
File: /contracts/StaderInsuranceFund.sol
44   if (address(this).balance < _amount || _amount == 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L44

```solidity
File: /contracts/StaderOracle.sol
532   if (_erChangeLimit == 0 || _erChangeLimit > ER_CHANGE_MAX_BPS) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L532

```solidity
File: /contracts/StaderStakePoolsManager.sol
171   if (assets > maxDeposit() || assets < minDeposit()) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L171

```solidity
File: /contracts/UserWithdrawalManager.sol
98    if (assets < staderConfig.getMinWithdrawAmount() || assets > staderConfig.getMaxWithdrawAmount()) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L98