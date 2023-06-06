# Gas Optimization

# Summary

| Number | Optimization Details                                                              | Context |
| :----: | :-------------------------------------------------------------------------------- | :-----: |
| [G-01] | With assembly, `.call` (bool success)  transfer can be done gas-optimized         |    6    |
| [G-02] | Avoid `contract` existence checks by using low level calls                        |   143   |
| [G-03] | Savings gas through `Variable` Declaration Outside Loops                          |   34    |
| [G-04] | Use assembly to check for `address(0)`                                            |    2    |
| [G-05] | `_ethXAmount` should be checked for 0 before calling a transfer                   |    1    |
| [G-06] | Duplicated `if()` statement checks should be refactored to a modifier or function |   10    |
| [G-07] | Use `nested if` and, avoid multiple check combinations &&                         |    8    |
| [G-08] | Use `assembly` to write `address storage` values                                  |    7    |
| [G-09] | Use `selfbalance()` instead of `address(this).balance`                            |   10    |
| [G-10] | Use hardcode address instead `address(this)`                                      |   16    |
| [G-11] | Use in constants instead of `type(uintx).max`                                     |    1    |
| [G-12] | Do not calculate `constants`                                                      |    1    |
| [G-13] | Using `> 0` costs less gas than `!= 0` when used on a unsigned integers           |    6    |
| [G-14] | Add Solidity's `optimizer`                                                        |    1    |

## [G-01] With assembly, `.call` (bool success)  transfer can be done `gas-optimized`

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

A good research of pashovkrum
https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

There are 6 instances of the topic.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L131

```solidity
File: contracts/Auction.sol

        // send the funds
131        (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');
132        if (!success) revert ETHWithdrawFailed();

```

```diff
File: contracts/Auction.sol


diff --git a/Auction.sol.origi b/Auction.sol
index 8280778..a05f5c4 100644
--- a/Auction.sol.origi
+++ b/Auction.sol
@@ -1,5 +1,8 @@
         lotItem.bids[msg.sender] -= withdrawalAmount;

         // send the funds
-        (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');
+        bool success;
+        assembly{
+               success := call(gas(), payable(msg.sender),withdrawalAmount,0,0)
+        }
         if (!success) revert ETHWithdrawFailed();

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L91
121

```solidity
File:   contracts/SocializingPool.sol

91        (bool success, ) = payable(staderConfig.getStaderTreasury()).call{value: _rewardsData.protocolETHRewards}('');
92        if (!success) {
93            revert ETHTransferFailed(staderConfig.getStaderTreasury(), _rewardsData.protocolETHRewards);
94        }
```

```diff
File:   contracts/SocializingPool.sol

diff --git a/Auction.sol.origi b/Auction.sol
index 4c0c7e9..93b942a 100644
--- a/Auction.sol.origi
+++ b/Auction.sol
@@ -2,8 +2,12 @@
             value: _rewardsData.userETHRewards
         }();

-        (bool success, ) = payable(staderConfig.getStaderTreasury()).call{value: _rewardsData.protocolETHRewards}('');
+         bool success;
+         assembly{
+                success := call(gas(), payable(staderConfig.getStaderTreasury()),_rewardsData.protocolETHRewards,0,0)
+                }
         if (!success) {
             revert ETHTransferFailed(staderConfig.getStaderTreasury(), _rewardsData.protocolETHRewards);
         }

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L48

```solidity
File: contracts/StaderInsuranceFund.sol

47          //slither-disable-next-line arbitrary-send-eth
48        (bool success, ) = payable(msg.sender).call{value: _amount}('');
49        if (!success) {
50            revert TransferFailed();
51        }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L102

```solidity
File: contracts/StaderStakePoolsManager.sol

102        (bool success, ) = payable(staderConfig.getUserWithdrawManager()).call{value: _amount}('');

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L229

```solidity
File: contracts/UserWithdrawalManager.sol

229        (bool success, ) = _recipient.call{value: _amount}('');

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L168

```solidity
File: contracts/library/UtilLib.sol

168        (bool success, ) = payable(_receiver).call{value: _amount}('');

```

## [G-02] Avoid `contract` existence checks by using low level calls 

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

Total Instances: `143`

Gas savings: `143 * 100 = 14300`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol

```solidity
File: contracts/Auction.sol

/// @audit transferFrom()
55        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {

/// @audit  transfer()
87        if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {


/// @audit  receiveEthFromAuction()
102        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveEthFromAuction{value: ethAmount}();


/// @audit  transfer()
114        if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol

```solidity
File: contracts/NodeELRewardVault.sol

/// @audit   poolId()
25        uint8 poolId = IVaultProxy(address(this)).poolId();

/// @audit   id()
26        uint256 operatorId = IVaultProxy(address(this)).id();

/// @audit  staderConfig()
27        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();


/// @audit  receiveExecutionLayerRewards()
36        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{value: userShare}();


/// @audit   depositFor()
40        IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
41            operator
42        );
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol

```solidity
File: contracts/Penalty.sol

/// @audit    getViolationsForValidator()
125        uint256[] memory violatedEpochs = IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot);

/// @audit    missedAttestationPenalty()
134            IStaderOracle(staderConfig.getStaderOracle()).missedAttestationPenalty(_pubkeyRoot) *

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

```solidity
File: contracts/PermissionedNodeRegistry.sol


/// @audit     poolAddressById()
113        if (IPoolUtils(poolUtils).poolAddressById(POOL_ID) != staderConfig.getPermissionedPool()) {


/// @audit    onlyValidName()
116        IPoolUtils(poolUtils).onlyValidName(_operatorName);


/// @audit    isExistingOperator()
125        if (IPoolUtils(poolUtils).isExistingOperator(msg.sender)) {


/// @audit    onlyValidKeys()
156            IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);


/// @audit    deployWithdrawVault()
157            address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(


/// @audit    transferETHOfDefectiveKeysToSSPM()
301            IPermissionedPool(permissionedPool).transferETHOfDefectiveKeysToSSPM(totalDefectedKeys);


/// @audit    fullDepositOnBeaconChain()
303        IPermissionedPool(permissionedPool).fullDepositOnBeaconChain(_readyToDepositPubkey);


/// @audit    settleFunds()
324            IValidatorWithdrawalVault(validatorRegistry[validatorId].withdrawVaultAddress).settleFunds();

/// @audit    onlyValidName()
402        IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);

/// @audit    hasEnoughSDCollateral()
709            !ISDCollateral(staderConfig.getSDCollateral()).hasEnoughSDCollateral(

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol

```solidity
File: contracts/PermissionedPool.sol

/// @audit      receiveExcessEthFromPool()
72        IStaderInsuranceFund(staderConfig.getStaderInsuranceFund()).reimburseUserFund(


/// @audit      receiveExcessEthFromPool()
79        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExcessEthFromPool{


/// @audit      validatorIdsByOperatorId()
114                uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdsByOperatorId(i, index);


/// @audit      updateQueuedValidatorIndex()
117            IPermissionedNodeRegistry(nodeRegistryAddress).updateQueuedValidatorIndex(


/// @audit      increaseTotalActiveValidatorCount()
125        IPermissionedNodeRegistry(nodeRegistryAddress).increaseTotalActiveValidatorCount(requiredValidators);


/// @audit      onlyPreDepositValidator()
138            IPermissionedNodeRegistry(nodeRegistryAddress).onlyPreDepositValidator(_pubkey[i]);


/// @audit    validatorIdByPubkey()
139            uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdByPubkey(_pubkey[i]);


/// @audit      getValidatorWithdrawCredential()
143            bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(


/// @audit      deposit()
155            IDepositContract(ethDepositContract).deposit{value: fullDepositSize}(


/// @audit      updateDepositStatusAndBlock()
161            IPermissionedNodeRegistry(nodeRegistryAddress).updateDepositStatusAndBlock(validatorId);


/// @audit      receiveExcessEthFromPool()
177        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExcessEthFromPool{


/// @audit      getTotalQueuedValidatorCount()
186        return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getTotalQueuedValidatorCount();


/// @audit      getTotalActiveValidatorCount()
193        return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getTotalActiveValidatorCount();


/// @audit      getOperatorTotalNonTerminalKeys()
205            INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getOperatorTotalNonTerminalKeys(


/// @audit      getCollateralETH
218        return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).getCollateralETH();

/// @audit      isExistingPubkey()
227        return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).isExistingPubkey(_pubkey);


/// @audit      isExistingOperator()
232        return INodeRegistry(staderConfig.getPermissionedNodeRegistry()).isExistingOperator(_operAddr);


/// @audit      getValidatorWithdrawCredential()
298        bytes memory withdrawCredential = IVaultFactory(_vaultFactory).getValidatorWithdrawCredential(


/// @audit      deposit()
310        IDepositContract(_ethDepositContract).deposit{value: preDepositSize}(


/// @audit      markValidatorStatusAsPreDeposit()
316        IPermissionedNodeRegistry(_nodeRegistryAddress).markValidatorStatusAsPreDeposit(pubkey);

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

```solidity
File: contracts/PermissionlessNodeRegistry.sol


/// @audit          poolAddressById()
95        if (IPoolUtils(poolUtils).poolAddressById(POOL_ID) != staderConfig.getPermissionlessPool()) {


/// @audit          poolAddressById()
98        IPoolUtils(poolUtils).poolAddressById(_operatorName);


/// @audit          isExistingOperator()
102        if (IPoolUtils(poolUtils).isExistingOperator(msg.sender)) {


/// @audit          getVaultFactory()
106        address nodeELRewardVault = IVaultFactory(staderConfig.getVaultFactory()).deployNodeELRewardVault(


/// @audit          onlyValidKeys()
141            IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);


/// @audit          deployWithdrawVault()
142            address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(


/// @audit          preDepositOnBeaconChain()
169        IPermissionlessPool(staderConfig.getPermissionlessPool()).preDepositOnBeaconChain{


/// @audit          depositFund()
210            IStaderInsuranceFund(staderConfig.getStaderInsuranceFund()).depositFund{


/// @audit          settleFunds()
254            IValidatorWithdrawalVault(validatorRegistry[validatorId].withdrawVaultAddress).settleFunds();


/// @audit          withdraw()
306                INodeELRewardVault(feeRecipientAddress).withdraw();


/// @audit          onlyValidName()
367        IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);


/// @audit          receiveRemainingCollateralETH()
394        IPermissionlessPool(staderConfig.getPermissionlessPool()).receiveRemainingCollateralETH{value: _amount}();


/// @audit          depositFor()
637        IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{


/// @audit          hasEnoughSDCollateral()
669            !ISDCollateral(staderConfig.getSDCollateral()).hasEnoughSDCollateral(

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol

```solidity
File: contracts/PermissionlessPool.sol

/// @audit      computeWithdrawVaultAddress()
95            address withdrawVault = IVaultFactory(vaultFactory).computeWithdrawVaultAddress(


/// @audit      POOL_ID()
96                INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID(),


/// @audit      getValidatorWithdrawCredential()
100            bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(withdrawVault);


/// @audit      deposit()
109            IDepositContract(staderConfig.getETHDepositContract()).deposit{value: staderConfig.getPreDepositSize()}(


/// @audit      transferCollateralToPool()
130        IPermissionlessNodeRegistry(nodeRegistryAddress).transferCollateralToPool(


/// @audit      nextQueuedValidatorIndex()
136        uint256 depositQueueStartIndex = IPermissionlessNodeRegistry(nodeRegistryAddress).nextQueuedValidatorIndex();


/// @audit      queuedValidators()
138            uint256 validatorId = IPermissionlessNodeRegistry(nodeRegistryAddress).queuedValidators(i);


/// @audit      updateNextQueuedValidatorIndex()
150        IPermissionlessNodeRegistry(nodeRegistryAddress).updateNextQueuedValidatorIndex(


/// @audit      increaseTotalActiveValidatorCount()
153        IPermissionlessNodeRegistry(nodeRegistryAddress).increaseTotalActiveValidatorCount(requiredValidators);


/// @audit      getTotalQueuedValidatorCount()
165        return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getTotalQueuedValidatorCount();


/// @audit      getTotalActiveValidatorCount()
172        return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getTotalActiveValidatorCount();


/// @audit      getOperatorTotalNonTerminalKeys()
184            INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getOperatorTotalNonTerminalKeys(


/// @audit      getCollateralETH()
192        return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).getCollateralETH();


/// @audit      isExistingPubkey()
201        return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).isExistingPubkey(_pubkey);


/// @audit      isExistingOperator()
206        return INodeRegistry(staderConfig.getPermissionlessNodeRegistry()).isExistingOperator(_operAddr);


/// @audit      getValidatorWithdrawCredential()
252        bytes memory withdrawCredential = IVaultFactory(_vaultFactoryAddress).getValidatorWithdrawCredential(


/// @audit      deposit()
262        IDepositContract(_ethDepositContract).deposit{value: _DEPOSIT_SIZE}(

/// @audit      updateDepositStatusAndBlock()
268        IPermissionlessNodeRegistry(_nodeRegistryAddress).updateDepositStatusAndBlock(_validatorId);

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L121

```solidity
File:  contracts/PoolSelector.sol


/// @audit    getPoolIdArray()
121        uint8[] memory poolIdArray = IPoolUtils(staderConfig.getPoolUtils()).getPoolIdArray();

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol

```solidity
File: contracts/PoolUtils.sol


/// @audit      protocolFee()
92        return IStaderPoolBase(poolAddressById[_poolId]).protocolFee();


/// @audit      operatorFee()
97        return IStaderPoolBase(poolAddressById[_poolId]).operatorFee();


/// @audit      getSocializingPoolAddress()
143        return IStaderPoolBase(poolAddressById[_poolId]).getSocializingPoolAddress();


/// @audit      getNodeRegistry()
163        return IStaderPoolBase(poolAddressById[_poolId]).getNodeRegistry();


/// @audit      POOL_ID()
283            INodeRegistry(IStaderPoolBase(_poolAddress).getNodeRegistry()).POOL_ID() != _poolId ||

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol

```solidity
File:  contracts/SDCollateral.sol

/// @audit       createLot
97        IAuction(staderConfig.getAuctionContract()).createLot(sdSlashed);

/// @audit       getSDPriceInETH
206        uint256 sdPriceInETH = IStaderOracle(staderConfig.getStaderOracle()).getSDPriceInETH();

/// @audit       getSDPriceInETH
211        uint256 sdPriceInETH = IStaderOracle(staderConfig.getStaderOracle()).getSDPriceInETH();

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol

```solidity
File:  contracts/SocializingPool.sol

/// @audit      balanceOf()
75            IERC20(staderConfig.getStaderToken()).balanceOf(address(this)) - totalOperatorSDRewardsRemaining


/// @audit      receiveExecutionLayerRewards()
87        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{


/// @audit      transfer()
129            if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L65

```solidity
File: contracts/StaderInsuranceFund.sol

/// @audit    receiveInsuranceFund()
65        IPermissionedPool(staderConfig.getPermissionedPool()).receiveInsuranceFund{value: _amount}();

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

```solidity
File: contracts/StaderOracle.sol

/// @audit      getSocializingPoolAddress()
256            address socializingPool = IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(

/// @audit    handleRewards()
259            ISocializingPool(socializingPool).handleRewards(_rewardsData);


/// @audit    withdrawnValidators()
436            INodeRegistry(_withdrawnValidators.nodeRegistry).withdrawnValidators(_withdrawnValidators.sortedPubkeys);


/// @audit    getSocializingPoolAddress()
577            IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(_poolId)


/// @audit    getSocializingPoolAddress()
608            ISocializingPool(IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(_poolId))

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol

```solidity
File:  contracts/StaderStakePoolsManager.sol

/// @audit      getExchangeRate()
129                IStaderOracle(staderConfig.getStaderOracle()).getExchangeRate().totalETHXSupply,


/// @audit      getExchangeRate()
136        return IStaderOracle(staderConfig.getStaderOracle()).getExchangeRate().totalETHBalance;


/// @audit      ethRequestedForWithdraw()
190            IUserWithdrawalManager(staderConfig.getUserWithdrawManager()).ethRequestedForWithdraw()


/// @audit      computePoolAllocationForDeposit()
197        uint256 selectedPoolCapacity = IPoolSelector(staderConfig.getPoolSelector()).computePoolAllocationForDeposit(


/// @audit      stakeUserETHToBeaconChain()
207        IStaderPoolBase(poolAddress).stakeUserETHToBeaconChain{value: selectedPoolCapacity * poolDepositSize}();


/// @audit      ethRequestedForWithdraw()
222            IUserWithdrawalManager(staderConfig.getUserWithdrawManager()).ethRequestedForWithdraw()


/// @audit      poolAllocationForExcessETHDeposit()
229        ).poolAllocationForExcessETHDeposit(availableETHForNewDeposit);


/// @audit      poolAddressById()
237            address poolAddress = IPoolUtils(poolUtils).poolAddressById(poolIdArray[i]);


/// @audit      getCollateralETH()
239                IPoolUtils(poolUtils).getCollateralETH(poolIdArray[i]);


/// @audit      stakeUserETHToBeaconChain()
243            IStaderPoolBase(poolAddress).stakeUserETHToBeaconChain{value: validatorToDeposit * poolDepositSize}();


/// @audit      getExchangeRate()
272        uint256 supply = IStaderOracle(staderConfig.getStaderOracle()).getExchangeRate().totalETHXSupply;


/// @audit      getExchangeRate()
295        uint256 supply = IStaderOracle(staderConfig.getStaderOracle()).getExchangeRate().totalETHXSupply;


/// @audit      mint()
321        ETHx(staderConfig.getETHxToken()).mint(_receiver, _shares);


/// @audit      getExchangeRate()
331                IStaderOracle(staderConfig.getStaderOracle()).getExchangeRate().totalETHXSupply == 0) && (!paused());

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol

```solidity
File: contracts/UserWithdrawalManager.sol


/// @audit      previewWithdraw()
97        uint256 assets = IStaderStakePoolManager(staderConfig.getStakePoolManager()).previewWithdraw(_ethXAmount);


/// @audit      safeMode()
118        if (IStaderOracle(staderConfig.getStaderOracle()).safeMode()) {


/// @audit      isVaultHealthy()
121        if (!IStaderStakePoolManager(staderConfig.getStakePoolManager()).isVaultHealthy()) {


/// @audit      getExchangeRate()
126        uint256 exchangeRate = IStaderStakePoolManager(poolManager).getExchangeRate();


/// @audit      burnFrom()
155            ETHx(staderConfig.getETHxToken()).burnFrom(address(this), lockedEthXToBurn);


/// @audit      transferETHToUserWithdrawManager()
156            IStaderStakePoolManager(poolManager).transferETHToUserWithdrawManager(ethToSendToFinalizeRequest);

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol

```solidity
File:  contracts/ValidatorWithdrawalVault.sol


/// @audit     poolId()
31        uint8 poolId = VaultProxy(payable(address(this))).poolId();


/// @audit     id()
32        uint256 validatorId = VaultProxy(payable(address(this))).id();


/// @audit     staderConfig()
33        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();


/// @audit     receiveWithdrawVaultUserShare()
46        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}();


/// @audit     depositFor()
48        IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(


/// @audit     poolId()
55        uint8 poolId = VaultProxy(payable(address(this))).poolId();


/// @audit     id()
56        uint256 validatorId = VaultProxy(payable(address(this))).id();


/// @audit     staderConfig()
57        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();


/// @audit     getNodeRegistry()
58        address nodeRegistry = IPoolUtils(staderConfig.getPoolUtils()).getNodeRegistry(poolId);


/// @audit     slashValidatorSD()
67            ISDCollateral(staderConfig.getSDCollateral()).slashValidatorSD(validatorId, poolId);


/// @audit     markValidatorSettled()
76        IPenalty(staderConfig.getPenaltyContract()).markValidatorSettled(poolId, validatorId);


/// @audit     receiveWithdrawVaultUserShare()
77        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}();


/// @audit     depositFor()
79        IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(


/// @audit     poolId()
94        uint8 poolId = VaultProxy(payable(address(this))).poolId();


/// @audit     staderConfig()
95        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();


/// @audit     calculateRewardShare()
118            ).calculateRewardShare(poolId, totalRewards);


/// @audit     getCollateralETH()
128        return IPoolUtils(_staderConfig.getPoolUtils()).getCollateralETH(_poolId);


/// @audit     getNodeRegistry()
144        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);


/// @audit     updateTotalPenaltyAmount()
148        IPenalty(_staderConfig.getPenaltyContract()).updateTotalPenaltyAmount(pubkeyArray);


/// @audit     totalPenaltyAmount()
149        return IPenalty(_staderConfig.getPenaltyContract()).totalPenaltyAmount(pubkey);

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol

```solidity
File:  library/UtilLib.sol

/// @audit      getNodeRegistry()
55        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);


/// @audit      getNodeRegistry()
71        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);


/// @audit      getNodeRegistry()
88        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);


/// @audit      getNodeRegistry()
100        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);


/// @audit      getNodeRegistry()
112        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);


/// @audit      getOperatorPoolId()
123        uint8 poolId = IPoolUtils(_staderConfig.getPoolUtils()).getOperatorPoolId(_operator);


/// @audit      getNodeRegistry()
124        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(poolId);


/// @audit      operatorIDByAddress()
125        uint256 operatorId = INodeRegistry(nodeRegistry).operatorIDByAddress(_operator);


/// @audit      getOperatorRewardAddress()
126        return INodeRegistry(nodeRegistry).getOperatorRewardAddress(operatorId);


/// @audit      getValidatorPoolId()
148        uint8 poolId = IPoolUtils(_staderConfig.getPoolUtils()).getValidatorPoolId(_pubkey);


/// @audit      getNodeRegistry()
149        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(poolId);


/// @audit      validatorIdByPubkey()
150        uint256 validatorId = INodeRegistry(nodeRegistry).validatorIdByPubkey(_pubkey);

```

## [G-03] Savings gas through `Variable` Declaration Outside Loops

When a variable is declared inside a loop, it is created and destroyed every time the loop iterates. This can be a computationally expensive process, especially if the loop iterates many times. By declaring a variable outside the loop, it is only created once and is reused throughout the loop

There are 34 instances of the topic.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

```solidity
File: contracts/PermissionedNodeRegistry.sol

/// @audit operator variable declare outside the loop
91        for (uint256 i = 0; i < permissionedNosLength; i++) {
92            address operator = _permissionedNOs[i];
93            UtilLib.checkNonZeroAddress(operator);


/// @audit  withdrawVault
157            address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(

/// @audit  validatorId declare outside the loop
272        for (uint256 i; i < frontRunValidatorsLength; ) {
273            uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];



/// @audit validatorId
285        for (uint256 i; i < invalidSignatureValidatorsLength; ) {
286            uint256 validatorId = validatorIdByPubkey[_invalidSignaturePubkey[i]];



/// @audit   validatorId
317        for (uint256 i; i < withdrawnValidatorCount; ) {
318:            uint256 validatorId = validatorIdByPubkey[_pubkeys[i]];
319            if (validatorRegistry[validatorId].status != ValidatorStatus.DEPOSITED) {
320                revert UNEXPECTED_STATUS();
321            }



534        for (uint256 i = _startIndex; i < _endIndex; ) {
535:            uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
536            if (isNonTerminalValidator(validatorId)) {


637        for (uint256 i = startIndex; i < endIndex; i++) {
638:            uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
639            validators[i - startIndex] = validatorRegistry[validatorId];
640        }


```

```diff
File: contracts/PermissionedNodeRegistry.sol

diff --git a/PermissionedNodeRegistry.sol.origi b/PermissionedNodeRegistry.sol
index ccc2fe1..49d1c2d 100644
--- a/PermissionedNodeRegistry.sol.origi
+++ b/PermissionedNodeRegistry.sol
@@ -1,7 +1,8 @@
         UtilLib.onlyManagerRole(msg.sender, staderConfig);
         uint256 permissionedNosLength = _permissionedNOs.length;
+        address operator;
         for (uint256 i = 0; i < permissionedNosLength; i++) {
-            address operator = _permissionedNOs[i];
+            operator = _permissionedNOs[i];
             UtilLib.checkNonZeroAddress(operator);
             permissionList[operator] = true;
             emit OperatorWhitelisted(operator);

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol

```solidity
File: contracts/PermissionedPool.sol


100        for (uint256 i = 1; i < selectedOperatorCapacityLength; ) {
101            uint256 validatorToDeposit = selectedOperatorCapacity[i];



/// @audit  validatorId decalare outside the loop
109            for (
110                uint256 index = nextQueuedValidatorIndex;
111                index < nextQueuedValidatorIndex + validatorToDeposit;
112                index++
113            ) {
114:       uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdsByOperatorId(i, 110index);



/// @audit   validatorId , withdrawCredential, fullDepositSize, depositDataRoot   variables decalare outside the loop

137        for (uint256 i; i < pubkeyCount; ) {
138            IPermissionedNodeRegistry(nodeRegistryAddress).onlyPreDepositValidator(_pubkey[i]);
139            uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdByPubkey(_pubkey[i]);
140            (, , , bytes memory depositSignature, address withdrawVaultAddress, , , ) = INodeRegistry(
141                nodeRegistryAddress
142            ).validatorRegistry(validatorId);
143            bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(
144                withdrawVaultAddress
145            );
146           uint256 fullDepositSize = staderConfig.getFullDepositSize();
147            bytes32 depositDataRoot = this.computeDepositDataRoot(

```

```diff
diff --git a/PermissionedPool.sol.origi b/PermissionedPool.sol
index 5d2943c..fcffe29 100644
--- a/PermissionedPool.sol.origi
+++ b/PermissionedPool.sol
@@ -1,16 +1,20 @@
         //decrease the preDeposit validator count
         decreasePreDepositValidatorCount(pubkeyCount);
+         uint256 validatorId ;
+         bytes memory withdrawCredential;
+         uint256 fullDepositSize;
+         bytes32 depositDataRoot;
         for (uint256 i; i < pubkeyCount; ) {
             IPermissionedNodeRegistry(nodeRegistryAddress).onlyPreDepositValidator(_pubkey[i]);
-            uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdByPubkey(_pubkey[i]);
+            validatorId = INodeRegistry(nodeRegistryAddress).validatorIdByPubkey(_pubkey[i]);
             (, , , bytes memory depositSignature, address withdrawVaultAddress, , , ) = INodeRegistry(
                 nodeRegistryAddress
             ).validatorRegistry(validatorId);
-            bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(
+            withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(
                 withdrawVaultAddress
             );
-            uint256 fullDepositSize = staderConfig.getFullDepositSize();
-            bytes32 depositDataRoot = this.computeDepositDataRoot(
+            fullDepositSize = staderConfig.getFullDepositSize();
+            depositDataRoot = this.computeDepositDataRoot(
                 _pubkey[i],
                 depositSignature,
                 withdrawCredential,


```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

```solidity
File: contracts/PermissionlessNodeRegistry.sol

142            address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(

200            uint256 validatorId = validatorIdByPubkey[_readyToDepositPubkey[i]];

216            uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];

226            uint256 validatorId = validatorIdByPubkey[_invalidSignaturePubkey[i]];

248            uint256 validatorId = validatorIdByPubkey[_pubkeys[i]];

416            uint256 validatorId = validatorIdsByOperatorId[operatorId][i];

544            uint256 validatorId = validatorIdsByOperatorId[operatorId][i];

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol

```solidity
File: contracts/PermissionlessPool.sol


95            address withdrawVault = IVaultFactory(vaultFactory).computeWithdrawVaultAddress(

100            bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(withdrawVault);

138            uint256 validatorId = IPermissionlessNodeRegistry(nodeRegistryAddress).queuedValidators(i);

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol

```solidity
File: contracts/PoolUtils.sol

169            address nodeRegistry = getNodeRegistry(poolIdArray[i]);

180            address nodeRegistry = getNodeRegistry(poolIdArray[i]);

191            address nodeRegistry = getNodeRegistry(poolIdArray[i]);

202            address nodeRegistry = getNodeRegistry(poolIdArray[i]);

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L486

```solidity
File: contracts/StaderOracle.sol

486                bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_mapd.sortedPubkeys[i]);

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol

```solidity
File: contracts/StaderStakePoolsManager.sol

233            uint256 validatorToDeposit = selectedPoolCapacity[i];

237            address poolAddress = IPoolUtils(poolUtils).poolAddressById(poolIdArray[i]);

238            uint256 poolDepositSize = staderConfig.getStakedEthPerNode() -

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol

```solidity
File: contracts/UserWithdrawalManager.sol

134            uint256 requiredEth = userWithdrawInfo.ethExpected;

135            uint256 lockedEthX = userWithdrawInfo.ethXAmount;

136            uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);

```

## [G-04] Use assembly to check for `address(0)`

Total Instances: `2`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L96

```solidity
File: contracts/UserWithdrawalManager.sol

96        if (_owner == address(0)) revert ZeroAddressReceived();

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L22

```solidity
File: contracts/library/UtilLib.sol

22        if (_address == address(0)) revert ZeroAddress();

```

## [G-05] `_ethXAmount` should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here:

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L104

```solidity
File:  contracts/UserWithdrawalManager.sol

104        IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);

```

```diff

diff --git a/UserWithdrawalManager.sol.origi b/UserWithdrawalManager.sol
index d00c589..56f3d8e 100644
--- a/UserWithdrawalManager.sol.origi
+++ b/UserWithdrawalManager.sol
@@ -1,5 +1,5 @@
         if (requestIdsByUserAddress[_owner].length + 1 > maxNonRedeemedUserRequestCount) {
             revert MaxLimitOnWithdrawRequestCountReached();
         }
-        IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);
+         if (_ethXAmount != 0)   IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);
         ethRequestedForWithdraw += assets;

```

## [G-06] Duplicated `if()` statement checks should be refactored to a modifier or function

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol

```solidity
File: contracts/Auction.sol

82         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
97         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
108        if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
122        if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

```solidity
File: contracts/PermissionedNodeRegistry.sol

207                if (!operatorStructById[i].active) {
227                if (!operatorStructById[i].active) {


340        if (!operatorStructById[_operatorId].active) {
725        if (!operatorStructById[_operatorId].active) {


590        if (_pageNumber == 0) {
625        if (_pageNumber == 0) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

```solidity
File: contracts/PermissionlessNodeRegistry.sol

496        if (_pageNumber == 0) {
531        if (_pageNumber == 0) {
565        if (_pageNumber == 0) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol

```solidity
File: contracts/PoolUtils.sol

170            if (INodeRegistry(nodeRegistry).isExistingPubkey(_pubkey)) {
203            if (INodeRegistry(nodeRegistry).isExistingPubkey(_pubkey)) {

181            if (INodeRegistry(nodeRegistry).isExistingOperator(_operAddr)) {
192            if (INodeRegistry(nodeRegistry).isExistingOperator(_operAddr)) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol

```solidity
File: contracts/SocializingPool.sol

92             if (!success) {
122            if (!success) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

```solidity
File: contracts/StaderOracle.sol

255        if ((submissionCount == trustedNodesCount / 2 + 1)) {
482        if ((submissionCount == trustedNodesCount / 2 + 1)) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol

```solidity
File: contracts/library/UtilLib.sol

59        if (_addr != withdrawVaultAddress) {
75        if (_addr != withdrawVaultAddress) {
90        if (_addr != withdrawVaultAddress) {

```

## [G-07] Use `nested if` and, avoid multiple check combinations &&

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

there are 8 instances

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L146

```solidity
File: contracts/SocializingPool.sol

145        for (uint256 i = 0; i < indexLength; i++) {
146            if (_amountSD[i] == 0 && _amountETH[i] == 0) {
147                revert InvalidAmount();
148            }
```

```diff
diff --git a/SocializingPool.sol.origi b/SocializingPool.sol
index 80707dc..4dac1ef 100644
--- a/SocializingPool.sol.origi
+++ b/SocializingPool.sol
@@ -1,7 +1,10 @@
     ) internal returns (uint256 _totalAmountSD, uint256 _totalAmountETH) {
         uint256 indexLength = _index.length;
         for (uint256 i = 0; i < indexLength; i++) {
-            if (_amountSD[i] == 0 && _amountETH[i] == 0) {
-                revert InvalidAmount();
+            if (_amountSD[i] == 0) {
+                if (_amountETH[i] == 0){
+                    revert InvalidAmount();
+                }
             }
             if (claimedRewards[_operator][_index[i]]) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L513

```solidity
File: contracts/StaderConfig.sol

512    function verifyDepositAndWithdrawLimits() internal view {
513        if (
514            !(variablesMap[MIN_DEPOSIT_AMOUNT] != 0 &&
515                variablesMap[MIN_WITHDRAW_AMOUNT] != 0 &&
516                variablesMap[MIN_DEPOSIT_AMOUNT] <= variablesMap[MAX_DEPOSIT_AMOUNT] &&
517                variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MAX_WITHDRAW_AMOUNT] &&
518                variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MIN_DEPOSIT_AMOUNT] &&
519                variablesMap[MAX_WITHDRAW_AMOUNT] >= variablesMap[MAX_DEPOSIT_AMOUNT])
520        ) {
521            revert InvalidLimits();
522        }
```

```diff

diff --git a/StaderConfig.sol.origi b/StaderConfig.sol
index f49ed44..8277988 100644
--- a/StaderConfig.sol.origi
+++ b/StaderConfig.sol
@@ -1,11 +1,14 @@
     function verifyDepositAndWithdrawLimits() internal view {
-        if (
-            !(variablesMap[MIN_DEPOSIT_AMOUNT] != 0 &&
-                variablesMap[MIN_WITHDRAW_AMOUNT] != 0 &&
-                variablesMap[MIN_DEPOSIT_AMOUNT] <= variablesMap[MAX_DEPOSIT_AMOUNT] &&
-                variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MAX_WITHDRAW_AMOUNT] &&
-                variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MIN_DEPOSIT_AMOUNT] &&
-                variablesMap[MAX_WITHDRAW_AMOUNT] >= variablesMap[MAX_DEPOSIT_AMOUNT])
-        ) {
-            revert InvalidLimits();
+        if (!(variablesMap[MIN_DEPOSIT_AMOUNT] != 0) {
+            if(variablesMap[MIN_WITHDRAW_AMOUNT] != 0 ) {
+                if(variablesMap[MIN_DEPOSIT_AMOUNT] <= variablesMap[MAX_DEPOSIT_AMOUNT]) {
+                    if(variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MAX_WITHDRAW_AMOUNT]) {
+                        if(variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MIN_DEPOSIT_AMOUNT]) {
+                            if(variablesMap[MAX_WITHDRAW_AMOUNT] >= variablesMap[MAX_DEPOSIT_AMOUNT]) {
+                                revert InvalidLimits();
+                            }
+                        }
+                    }
+                }
+            }
         }

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

```solidity
File: contracts/StaderOracle.sol

147        if (
148            submissionCount == trustedNodesCount / 2 + 1 &&
149            _exchangeRate.reportingBlockNumber > exchangeRate.reportingBlockNumber
150        ) {



186        if (
187            !staderConfig.onlyManagerRole(msg.sender) &&
188            erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY > block.number
189        ) {



371        if (
372            submissionCount == trustedNodesCount / 2 + 1 &&
373            _validatorStats.reportingBlockNumber > validatorStats.reportingBlockNumber
374        ) {


431        if (
432            submissionCount == trustedNodesCount / 2 + 1 &&
433            _withdrawnValidators.reportingBlockNumber > reportingBlockNumberForWithdrawnValidators
434        ) {



664        if (
665            !(newExchangeRate >= (currentExchangeRate * (ER_CHANGE_MAX_BPS - erChangeLimit)) / ER_CHANGE_MAX_BPS &&
666                newExchangeRate <= ((currentExchangeRate * (ER_CHANGE_MAX_BPS + erChangeLimit)) / ER_CHANGE_MAX_BPS))
667        ) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L35

```solidity
File: contracts/ValidatorWithdrawalVault.sol

35        if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {

```

## [G-08] Use `assembly` to write `address storage` values

There are 7 instances of the topic.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol

```solidity
File: contracts/Penalty.sol

43        ratedOracleAddress = _ratedOracleAddress;

86        ratedOracleAddress = _ratedOracleAddress;

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L41

```solidity
File: OperatorRewardsCollector.sol

41        balances[_receiver] += msg.value;

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol

```solidity
File: contracts/PoolUtils.sol

45        poolAddressById[_poolId] = _poolAddress;

63        poolAddressById[_poolId] = _newPoolAddress;

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L70

```solidity
File: contracts/VaultProxy.sol

70        owner = _owner;

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L95

```solidity
File: factory/VaultFactory.sol


93   function updateVaultProxyAddress(address _vaultProxyImpl) external override onlyRole(DEFAULT_ADMIN_ROLE) {
94        UtilLib.checkNonZeroAddress(_vaultProxyImpl);
95        vaultProxyImplementation = _vaultProxyImpl;
96        emit UpdatedVaultProxyImplementation(vaultProxyImplementation);
97    }
```

### Recommendation Code

```diff
diff --git a/VaultFactory.sol.origi b/VaultFactory.sol
index cf3c804..3f3892f 100644
--- a/VaultFactory.sol.origi
+++ b/VaultFactory.sol
@@ -1,6 +1,8 @@
     //update the implementation address of vaultProxy contract
     function updateVaultProxyAddress(address _vaultProxyImpl) external override onlyRole(DEFAULT_ADMIN_ROLE) {
         UtilLib.checkNonZeroAddress(_vaultProxyImpl);
-        vaultProxyImplementation = _vaultProxyImpl;
+        assembly {
+            sstore(vaultProxyImplementation.slot, _vaultProxyImpl)
+        }
         emit UpdatedVaultProxyImplementation(vaultProxyImplementation);
     }

```

## [G-09] Use `selfbalance()` instead of `address(this).balance`

`address(this).balance` retrieves the current balance of the contract address. This operation requires a read operation on the blockchain, which can incur gas costs. On the other hand, `selfbalance()` is a built-in Solidity function that retrieves the balance of the current contract address without requiring a read operation on the blockchain, thus potentially saving gas.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L28

```solidity
File: contracts/NodeELRewardVault.sol

28        uint256 totalRewards = address(this).balance;

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol

```solidity
File: contracts/PermissionedPool.sol

174        if (preDepositValidatorCount != 0 || address(this).balance == 0) {

178            value: address(this).balance

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L69

```solidity
File: contracts/SocializingPool.sol

69            address(this).balance - totalOperatorETHRewardsRemaining

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol

```solidity
File: contracts/StaderInsuranceFund.sol

43        if (address(this).balance < _amount || _amount == 0) {

62        if (address(this).balance < _amount) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol

```solidity
File: contracts/StaderStakePoolsManager.sol

189            address(this).balance,

221            address(this).balance,

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L224

```solidity
File: contracts/UserWithdrawalManager.sol

224        if (address(this).balance < _amount) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol

```solidity
File: contracts/ValidatorWithdrawalVault.sol

34        uint256 totalRewards = address(this).balance;

90        uint256 contractBalance = address(this).balance;

```

## [G-10] Use hardcode address instead `address(this)`

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's script.sol and solmate's LibRlp.sol contracts can help achieve this.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

https://twitter.com/transmissions11/status/1518507047943245824

there are `16` instances

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L55

```solidity
File: contracts/Auction.sol

55        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol

```solidity
File:  contracts/NodeELRewardVault.sol

25        uint8 poolId = IVaultProxy(address(this)).poolId();

26        uint256 operatorId = IVaultProxy(address(this)).id();

27        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L47

```solidity
File:  contracts/SDCollateral.sol

47        if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L75

```solidity
File: contracts/SocializingPool.sol

75            IERC20(staderConfig.getStaderToken()).balanceOf(address(this)) - totalOperatorSDRewardsRemaining

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L104

```solidity
File: contracts/UserWithdrawalManager.sol


104        IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);

155            ETHx(staderConfig.getETHxToken()).burnFrom(address(this), lockedEthXToBurn);

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L31

```solidity
File:  contracts/ValidatorWithdrawalVault.sol


31        uint8 poolId = VaultProxy(payable(address(this))).poolId();


32        uint256 validatorId = VaultProxy(payable(address(this))).id();


33        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();


55        uint8 poolId = VaultProxy(payable(address(this))).poolId();


56        uint256 validatorId = VaultProxy(payable(address(this))).id();


57        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();


94        uint8 poolId = VaultProxy(payable(address(this))).poolId();


95        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();

```

## [G-11] Use in constants instead of `type(uintx).max`

`type(uin256).max` it uses more gas in the distribution process and also for each transaction than constant usage.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L106

```solidity
File:  contracts/SDCollateral.sol

106        IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);

```

## [G-12] Do not calculate `constants`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L26

```solidity
File:  contracts/StaderOracle.sol

26    uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days

```

## [G-13] Using `> 0` costs less gas than `!= 0` when used on a unsigned integers

This change saves 3 gas per instance

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

```solidity
File:  contracts/PermissionedNodeRegistry.sol

/// @audit  validatorPerOperator is unsigned integer
204        bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);


/// @audit  totalDefectedKeys is unsigned integer
299        if (totalDefectedKeys > 0) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L209

```solidity
File:  contracts/PermissionlessNodeRegistry.sol

209        if (frontRunValidatorsLength > 0) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L119

```solidity
File:  contracts/SocializingPool.sol

119        if (totalAmountETH > 0) {

127        if (totalAmountETH > 0) {

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L115

```solidity
File:  contracts/ValidatorWithdrawalVault.sol

115        if (totalRewards > 0) {

```

## [G-14] Add Solidity's `optimizer`

Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.

https://github.com/code-423n4/2023-06-stader/blob/main/foundry.toml

```javascript
File: 2023 - 06 - stader / blob / main / foundry.toml;

[profile.default];
src = "contracts";
out = "out";
libs = ["node_modules", "lib"];
solc_version = "0.8.16";
test = "test";
cache_path = "cache_forge";
```

```javascript
File: 2023 - 06 - stader / blob / main / foundry.toml;

[profile.default]
src = 'contracts'
out = 'out'
libs = ['node_modules', 'lib']
solc_version="0.8.16"
test = 'test'
cache_path  = 'cache_forge'


+   optimizer = true
+   optimizer_runs = 200

```
