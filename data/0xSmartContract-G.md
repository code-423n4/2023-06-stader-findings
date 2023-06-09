###  Gas Optimizations List
 --------------------------------------------------
- [x] **Gas 01** → Structs can be packed into fewer storage slots
- [x] **Gas 02** → With assembly, `.call (bool success)` transfer can be done gas-optimized
- [x] **Gas 03** → if () / require() statements that check input arguments should be at the top of the function
- [x] **Gas 04** → Gas overflow during iteration (DoS)
- [x] **Gas 05** → Reduce Gas Costs by Emitting Events Outside of For Loops
- [x] **Gas 06** → Use ``assembly`` to write _address storage values_
- [x] **Gas 07** → Duplicated require()/if() checks should be refactored to a modifier or function
- [x] **Gas 08** → Use ``assembly`` to write _address storage values_
- [x] **Gas 09** → Using ``delete``  instead of setting mapping/struct ``0`` saves gas
- [x] **Gas 10** → Do not calculate constants variables



### [G-01] Structs can be packed into fewer storage slots

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage ~20000 gas.

Note: These are added to this list because they are not captured in automatic finds.

Below are the details of packaging structs. 

6 results - 2 files:

|Title | Total Gas Saved | Instance|
|:--:|:-------:|:--:|
|Struct packed | 6k | 3 |

1. The ``UserWithdrawInfo`` structure can be packaged as suggested below  (from 5 slots to 4 slots)

Since the ``uint256 requestBlock`` variable in this struct represents the block number, if it is updated to ``uint96``, 1 slot will be gained.

    **1 slot win: 1 * 2k  =  2k gas saved**

```solidity
contracts\UserWithdrawalManager.sol:

  40      /// @notice structure representing a user request for withdrawal.
  41:     struct UserWithdrawInfo {
  42:         address payable owner; // address that can claim eth on behalf of this request
  43:         uint256 ethXAmount; //amount of ethX share locked for withdrawal
  44:         uint256 ethExpected; //eth requested according to given share and exchangeRate
  45:         uint256 ethFinalized; // final eth for claiming according to finalize exchange rate
  46:         uint256 requestBlock; // block number of withdraw request
  47:     }
  48

```

```diff
contracts\UserWithdrawalManager.sol:

  40      /// @notice structure representing a user request for withdrawal.
  41:     struct UserWithdrawInfo {
  42:         address payable owner; // address that can claim eth on behalf of this request
+ 46:         uint96 requestBlock; // block number of withdraw request
  43:         uint256 ethXAmount; //amount of ethX share locked for withdrawal
  44:         uint256 ethExpected; //eth requested according to given share and exchangeRate
  45:         uint256 ethFinalized; // final eth for claiming according to finalize exchange rate
- 46:         uint256 requestBlock; // block number of withdraw request
  47:     }
  48
  
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L41-L48


2. The ``ValidatorStats` structure can be packaged as suggested below  (from 3 slots to 2 slots)

```solidity
contracts\StaderOracle.sol:

   7: import './interfaces/IStaderOracle.sol';
 
  24:     ValidatorStats public validatorStats;

```
    **1 slot win: 1 * 2k  =  2k gas saved**

```solidity
contracts\interfaces\IStaderOracle.sol:
  42  /// @notice This struct holds statistics related to validators in the beaconchain.
  43: struct ValidatorStats {
  44:     /// @notice The block number when the validator stats was last updated.
  45:     uint256 reportingBlockNumber;
  46:     /// @notice The total balance of all exiting validators.
  47:     uint128 exitingValidatorsBalance;
  48:     /// @notice The total balance of all exited validators.
  49:     uint128 exitedValidatorsBalance;
  50:     /// @notice The total balance of all slashed validators.
  51:     uint128 slashedValidatorsBalance;
  52:     /// @notice The number of currently exiting validators.
  53:     uint32 exitingValidatorsCount;
  54:     /// @notice The number of validators that have exited.
  55:     uint32 exitedValidatorsCount;
  56:     /// @notice The number of validators that have been slashed.
  57:     uint32 slashedValidatorsCount;
  58: }
  59  

```


```diff
contracts\interfaces\IStaderOracle.sol:
  42  /// @notice This struct holds statistics related to validators in the beaconchain.
  43: struct ValidatorStats {
- 44:     /// @notice The block number when the validator stats was last updated.
- 45:     uint256 reportingBlockNumber;
  46:     /// @notice The total balance of all exiting validators.
  47:     uint128 exitingValidatorsBalance;
  48:     /// @notice The total balance of all exited validators.
  49:     uint128 exitedValidatorsBalance;
  50:     /// @notice The total balance of all slashed validators.
  51:     uint128 slashedValidatorsBalance;
  52:     /// @notice The number of currently exiting validators.
  53:     uint32 exitingValidatorsCount;
  54:     /// @notice The number of validators that have exited.
  55:     uint32 exitedValidatorsCount;
  56:     /// @notice The number of validators that have been slashed.
  57:     uint32 slashedValidatorsCount;
+ 44:     /// @notice The block number when the validator stats was last updated.
+ 45:     uint32 reportingBlockNumber;
  58: }
  59  
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L24
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/interfaces/IStaderOracle.sol#L43-L58



3. The `` RewardsData ` structure can be packaged as suggested below  (from 8 slots to 7 slots)
```solidity
contracts\SocializingPool.sol:

   7: import './interfaces/ISocializingPool.sol';

  31:     RewardsData public lastReportedRewardsData;

  32:     mapping(uint256 => RewardsData) public rewardsDataMap;
 
```

    **1 slot win: 1 * 2k  =  2k gas saved**

```solidity
contracts\interfaces\ISocializingPool.sol:
   8  /// @notice This struct holds rewards merkleRoot and rewards split
   9: struct RewardsData {
  10:     /// @notice The block number when the rewards data was last updated
  11:     uint256 reportingBlockNumber;
  12:     /// @notice The index of merkle tree or rewards cycle
  13:     uint256 index;
  14:     /// @notice The merkle root hash
  15:     bytes32 merkleRoot;
  16:     /// @notice pool id of operators
  17:     uint8 poolId;
  18:     /// @notice operator ETH rewards for index cycle
  19:     uint256 operatorETHRewards;
  20:     /// @notice user ETH rewards for index cycle
  21:     uint256 userETHRewards;
  22:     /// @notice protocol ETH rewards for index cycle
  23:     uint256 protocolETHRewards;
  24:     /// @notice operator SD rewards for index cycle
  25:     uint256 operatorSDRewards;
  26: }

```


```diff
contracts\interfaces\ISocializingPool.sol:
   8  /// @notice This struct holds rewards merkleRoot and rewards split
   9: struct RewardsData {
  10:     /// @notice The block number when the rewards data was last updated
- 11:     uint256 reportingBlockNumber;
+ 11:     uint64 reportingBlockNumber;
+ 17:     uint8 poolId;
  12:     /// @notice The index of merkle tree or rewards cycle
  13:     uint256 index;
  14:     /// @notice The merkle root hash
  15:     bytes32 merkleRoot;
  16:     /// @notice pool id of operators
- 17:     uint8 poolId;
  18:     /// @notice operator ETH rewards for index cycle
  19:     uint256 operatorETHRewards;
  20:     /// @notice user ETH rewards for index cycle
  21:     uint256 userETHRewards;
  22:     /// @notice protocol ETH rewards for index cycle
  23:     uint256 protocolETHRewards;
  24:     /// @notice operator SD rewards for index cycle
  25:     uint256 operatorSDRewards;
  26: }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/interfaces/ISocializingPool.sol#L9-L26
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L31-LL32


### [G-02] With assembly, `.call (bool success)` transfer can be done gas-optimized

`return` data `(bool success,)` has to be stored due to EVM architecture, but in a usage like below, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

7 results - 6 files:

```solidity
contracts\Auction.sol:

  130          // send the funds
  131:         (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');
  132          if (!success) revert ETHWithdrawFailed();

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L131-L132


```solidity
contracts\SocializingPool.sol:

  91:         (bool success, ) = payable(staderConfig.getStaderTreasury()).call{value: _rewardsData.protocolETHRewards}('');
  92:         if (!success) {
  93:             revert ETHTransferFailed(staderConfig.getStaderTreasury(), _rewardsData.protocolETHRewards);
  94:         }



  121:             (success, ) = payable(operatorRewardsAddr).call{value: totalAmountETH}('');
  122:             if (!success) {
  123:                 revert ETHTransferFailed(operatorRewardsAddr, totalAmountETH);
  124:             }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L91-L94
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L121-L124


```solidity
contracts\StaderInsuranceFund.sol:

  47:         //slither-disable-next-line arbitrary-send-eth
  48:         (bool success, ) = payable(msg.sender).call{value: _amount}('');
  49:         if (!success) {
  50:             revert TransferFailed();
  51:         }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L48-L51



```solidity
contracts\StaderStakePoolsManager.sol:

  101:         //slither-disable-next-line arbitrary-send-eth
  102:         (bool success, ) = payable(staderConfig.getUserWithdrawManager()).call{value: _amount}('');
  103:         if (!success) {
  104:             revert TransferFailed();
  105:         }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L102-L105


```solidity
contracts\UserWithdrawalManager.sol:
  227  
  228:         //slither-disable-next-line arbitrary-send-eth
  229:         (bool success, ) = _recipient.call{value: _amount}('');
  230:         if (!success) {
  231:             revert ETHTransferFailed();
  232:         }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L228-L232


```solidity
contracts\library\UtilLib.sol:

  168:         (bool success, ) = payable(_receiver).call{value: _amount}('');
  169:         if (!success) {
  170:             revert TransferFailed();
  171:         }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L168-L171



**Recommendation Code:**
```diff
contracts\library\UtilLib.sol:

- 168:         (bool success, ) = payable(_receiver).call{value: _amount}('');
+              address addr = payable(_receiver)
+              bool success;
+              assembly {
+              sent := call(gas(), _receiver, _amount, 0, 0, 0, 0)
+              }
  169:         if (!success) {
  170:             revert TransferFailed();
  171:         }

```


### [G-03] if () / require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

 4 results - 4 files:

```solidity
contracts\NodeELRewardVault.sol:
  23  
  24:     function withdraw() external override {
  25:         uint8 poolId = IVaultProxy(address(this)).poolId();
  26:         uint256 operatorId = IVaultProxy(address(this)).id();
  27:         IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
  28:         uint256 totalRewards = address(this).balance;
  29:         if (totalRewards == 0) {
  30:             revert NotEnoughRewardToWithdraw();
  31:         }
  32:         (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
  33:             .calculateRewardShare(poolId, totalRewards);
  34: 
  35:         // Distribute rewards
  36:         IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{value: userShare}();
  37:         // slither-disable-next-line arbitrary-send-eth
  38:         UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
  39:         address operator = UtilLib.getOperatorAddressByOperatorId(poolId, operatorId, staderConfig);
  40:         IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
  41:             operator
  42:         );
  43: 
  44:         emit Withdrawal(protocolShare, operatorShare, userShare);
  45:     }
  46  }

```

```diff
contracts\NodeELRewardVault.sol:
  23  
  24:     function withdraw() external override {
+ 28:         uint256 totalRewards = address(this).balance;
+ 29:         if (totalRewards == 0) {
+ 30:             revert NotEnoughRewardToWithdraw();
+ 31:         }
  25:         uint8 poolId = IVaultProxy(address(this)).poolId();
  26:         uint256 operatorId = IVaultProxy(address(this)).id();
  27:         IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
- 28:         uint256 totalRewards = address(this).balance;
- 29:         if (totalRewards == 0) {
- 30:             revert NotEnoughRewardToWithdraw();
- 31:         }
  32:         (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
  33:             .calculateRewardShare(poolId, totalRewards);
  34: 
  35:         // Distribute rewards
  36:         IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{value: userShare}();
  37:         // slither-disable-next-line arbitrary-send-eth
  38:         UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
  39:         address operator = UtilLib.getOperatorAddressByOperatorId(poolId, operatorId, staderConfig);
  40:         IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
  41:             operator
  42:         );
  43: 
  44:         emit Withdrawal(protocolShare, operatorShare, userShare);
  45:     }
  46  }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L24-L46


```solidity
contracts\Auction.sol:
   93  
   94:     function transferHighestBidToSSPM(uint256 lotId) external override nonReentrant {
   95:         LotItem storage lotItem = lots[lotId];
   96:         uint256 ethAmount = lotItem.highestBidAmount;
   97: 
   98:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
   99:         if (ethAmount == 0) revert NoBidPlaced();
  100:         if (lotItem.ethExtracted) revert AlreadyClaimed();
  101: 
  102:         lotItem.ethExtracted = true;
  103:         IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveEthFromAuction{value: ethAmount}();
  104:         emit ETHClaimed(lotId, staderConfig.getStakePoolManager(), ethAmount); //@audit don't use externalcall
  105:     }
  106  

```

```diff
contracts\Auction.sol:
   93  
   94:     function transferHighestBidToSSPM(uint256 lotId) external override nonReentrant {
   95:         LotItem storage lotItem = lots[lotId];
+  98:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
   96:         uint256 ethAmount = lotItem.highestBidAmount;
   97: 
-  98:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
   99:         if (ethAmount == 0) revert NoBidPlaced();
  100:         if (lotItem.ethExtracted) revert AlreadyClaimed();
  101: 
  102:         lotItem.ethExtracted = true;
  103:         IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveEthFromAuction{value: ethAmount}();
  104:         emit ETHClaimed(lotId, staderConfig.getStakePoolManager(), ethAmount); //@audit don't use externalcall
  105:     }
  106  

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L97


Since the ``getValidatorsByOperator()`` function in the **PermissionedNodeRegistry.sol** contract and the ``getValidatorsByOperator()`` function in the **PermissionlessNodeRegistry.sol** contract are exactly the same, this gas optimization is in the two functions of both contracts. valid and sampling is only for `PermissionlessNodeRegistry.sol``.

```solidity
contracts\PermissionlessNodeRegistry.sol:
  526:     function getValidatorsByOperator(
  527:         address _operator,
  528:         uint256 _pageNumber,
  529:         uint256 _pageSize
  530:     ) external view override returns (Validator[] memory) {
  531:         if (_pageNumber == 0) {
  532:             revert PageNumberIsZero();
  533:         }
  534:         uint256 startIndex = (_pageNumber - 1) * _pageSize;
  535:         uint256 endIndex = startIndex + _pageSize;
  536:         uint256 operatorId = operatorIDByAddress[_operator];
  537:         if (operatorId == 0) {
  538:             revert OperatorNotOnBoarded();
  539:         }
  540:         uint256 validatorCount = getOperatorTotalKeys(operatorId);
  541:         endIndex = endIndex > validatorCount ? validatorCount : endIndex;
  542:         Validator[] memory validators = new Validator[](endIndex > startIndex ? endIndex - startIndex : 0);
  543:         for (uint256 i = startIndex; i < endIndex; i++) {
  544:             uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
  545:             validators[i - startIndex] = validatorRegistry[validatorId];
  546:         }
  547: 
  548:         return validators;
  549:     }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L526-L549


```solidity
contracts\PermissionedNodeRegistry.sol:
  619       */
  620:     function getValidatorsByOperator(
  621:         address _operator,
  622:         uint256 _pageNumber,
  623:         uint256 _pageSize
  624:     ) external view override returns (Validator[] memory) {
  625:         if (_pageNumber == 0) {
  626:             revert PageNumberIsZero();
  627:         }
  628:         uint256 startIndex = (_pageNumber - 1) * _pageSize;
  629:         uint256 endIndex = startIndex + _pageSize;
  630:         uint256 operatorId = operatorIDByAddress[_operator];
  631:         if (operatorId == 0) {
  632:             revert OperatorNotOnBoarded();
  633:         }
  634:         uint256 validatorCount = getOperatorTotalKeys(operatorId);
  635:         endIndex = endIndex > validatorCount ? validatorCount : endIndex;
  636:         Validator[] memory validators = new Validator[](endIndex > startIndex ? endIndex - startIndex : 0);
  637:         for (uint256 i = startIndex; i < endIndex; i++) {
  638:             uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
  639:             validators[i - startIndex] = validatorRegistry[validatorId];
  640:         }
  641: 
  642:         return validators;
  643:     }
  644  

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L620-L644


```diff
contracts\PermissionedNodeRegistry.sol:
  619       */
  620:     function getValidatorsByOperator(
  621:         address _operator,
  622:         uint256 _pageNumber,
  623:         uint256 _pageSize
  624:     ) external view override returns (Validator[] memory) {
  625:         if (_pageNumber == 0) {
  626:             revert PageNumberIsZero();
  627:         }
+ 630:         uint256 operatorId = operatorIDByAddress[_operator];
+ 631:         if (operatorId == 0) {
+ 632:             revert OperatorNotOnBoarded();
+ 633:         }
  628:         uint256 startIndex = (_pageNumber - 1) * _pageSize;
  629:         uint256 endIndex = startIndex + _pageSize;
- 630:         uint256 operatorId = operatorIDByAddress[_operator];
- 631:         if (operatorId == 0) {
- 632:             revert OperatorNotOnBoarded();
- 633:         }
  634:         uint256 validatorCount = getOperatorTotalKeys(operatorId);
  635:         endIndex = endIndex > validatorCount ? validatorCount : endIndex;
  636:         Validator[] memory validators = new Validator[](endIndex > startIndex ? endIndex - startIndex : 0);
  637:         for (uint256 i = startIndex; i < endIndex; i++) {
  638:             uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
  639:             validators[i - startIndex] = validatorRegistry[validatorId];
  640:         }
  641: 
  642:         return validators;
  643:     }
  644  

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L620-L643


### [G-04] Gas overflow during iteration (DoS)

Each iteration of the cycle requires a gas flow. There may come a time when more gas than allocated is required to save a block. In this case, all iterations of the loop will fail.

There are 14 examples of the topic. The suggestion block is made for only one function.

14 results - 9 files:

```solidity
contracts\Penalty.sol:

   98:     function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
   99:         uint256 reportedValidatorCount = _pubkey.length;
  100:         for (uint256 i; i < reportedValidatorCount; ) {

```

**Recommendation Code:**

```diff
contracts\Penalty.sol:

+  uint256 private constant MAXPUBKEYLENGTH = 333; 

   98:     function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
   99:         uint256 reportedValidatorCount = _pubkey.length;
+              if (reportedValidatorCount > MAXPUBKEYLENGTH) revert maxLength();
  100:         for (uint256 i; i < reportedValidatorCount; ) {

```

```solidity
contracts\PermissionedNodeRegistry.sol:

  88:     function whitelistPermissionedNOs(address[] calldata _permissionedNOs) external override {
  89:         UtilLib.onlyManagerRole(msg.sender, staderConfig);
  90:         uint256 permissionedNosLength = _permissionedNOs.length;
  91:         for (uint256 i = 0; i < permissionedNosLength; i++) {


  254:     function markValidatorReadyToDeposit(
  256:         bytes[] calldata _frontRunPubkey,
  258:     ) external override nonReentrant whenNotPaused {
  // other codes...
  261:         uint256 frontRunValidatorsLength = _frontRunPubkey.length;
  // other codes...
  272:         for (uint256 i; i < frontRunValidatorsLength; ) {
  273:             uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];
  274              // only PRE_DEPOSIT status check will also include validatorId = 0 check
  311:     function withdrawnValidators(bytes[] calldata _pubkeys) external override {

```


```solidity
contracts\PermissionedPool.sol:

  129:     function fullDepositOnBeaconChain(bytes[] calldata _pubkey) external nonReentrant {
  // other codes...
  134:         uint256 pubkeyCount = _pubkey.length;
  // other codes...
  137:         for (uint256 i; i < pubkeyCount; ) {

```


```solidity
contracts\PermissionlessNodeRegistry.sol:

  183:     function markValidatorReadyToDeposit(
  184:         bytes[] calldata _readyToDepositPubkey,
  185:         bytes[] calldata _frontRunPubkey,
  186:         bytes[] calldata _invalidSignaturePubkey
  187:     ) external override nonReentrant whenNotPaused {

  189:         uint256 readyToDepositValidatorsLength = _readyToDepositPubkey.length;
  190:         uint256 frontRunValidatorsLength = _frontRunPubkey.length;
  191:         uint256 invalidSignatureValidatorsLength = _invalidSignaturePubkey.length;
  // other codes...
  199:         for (uint256 i; i < readyToDepositValidatorsLength; ) {
  // other codes...
  215:         for (uint256 i; i < frontRunValidatorsLength; ) {
  // other codes...
  225:         for (uint256 i; i < invalidSignatureValidatorsLength; ) {



  241:     function withdrawnValidators(bytes[] calldata _pubkeys) external override {
  // other codes...
  243:         uint256 withdrawnValidatorCount = _pubkeys.length;
  // other codes...
  247:         for (uint256 i; i < withdrawnValidatorCount; ) {

```


```solidity
contracts\PermissionedNodeRegistry.sol:

  311:     function withdrawnValidators(bytes[] calldata _pubkeys) external override {
  // other codes...
  313:         uint256 withdrawnValidatorCount = _pubkeys.length;
  // other codes...
  317:         for (uint256 i; i < withdrawnValidatorCount; ) {

```


```solidity
contracts\PoolSelector.sol:

  119:     function updatePoolWeights(uint256[] calldata _poolTargets) external {
  // other codes...
  123:         uint256 poolTargetLength = _poolTargets.length;
  // other codes...
  130:         for (uint256 i = 0; i < poolTargetLength; i++) {
  131              totalWeight += _poolTargets[i];

```


```solidity
contracts\PoolUtils.sol:

  72:     function processValidatorExitList(bytes[] calldata _pubkeys) external override {
  73:         UtilLib.onlyOperatorRole(msg.sender, staderConfig);
  74:         uint256 exitValidatorCount = _pubkeys.length;
  75:         for (uint256 i; i < exitValidatorCount; ) {
  76              emit ExitValidator(_pubkeys[i]);

```


```solidity
contracts\SocializingPool.sol:

  137:     function _claim(
  138:         uint256[] calldata _index,
  // other codes...
  143:     ) internal returns (uint256 _totalAmountSD, uint256 _totalAmountETH) {
  144:         uint256 indexLength = _index.length;
  145:         for (uint256 i = 0; i < indexLength; i++) {

```


### [G-05] Reduce Gas Costs by Emitting Events Outside of For Loops

Placing events inside loops results in the event being emitted at each iteration, which can significantly increase gas consumption. By moving events outside the loop and emitting them globally, unnecessary gas expenses can be minimized, leading to more efficient and cost-effective contract execution.

15 results - 7 files:

```solidity
contracts\PermissionedNodeRegistry.sol:

  88:     function whitelistPermissionedNOs(address[] calldata _permissionedNOs) external override {
  // other codes...
  91:         for (uint256 i = 0; i < permissionedNosLength; i++) {
  92:             address operator = _permissionedNOs[i];
  93:             UtilLib.checkNonZeroAddress(operator);
  94:             permissionList[operator] = true;
  95:             emit OperatorWhitelisted(operator);
  96:         }



  140:     function addValidatorKeys(
  // other codes...
  155:         for (uint256 i; i < keyCount; ) {
  // other codes...
  175:             emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId); //@audit nextValidatorId?



  254:     function markValidatorReadyToDeposit(
  // other codes...
  272:         for (uint256 i; i < frontRunValidatorsLength; ) {
  273:             uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];
  274:             // only PRE_DEPOSIT status check will also include validatorId = 0 check
  275:             // as status for that will be INITIALIZED(default status)
  276:             onlyPreDepositValidator(validatorId);
  277:             handleFrontRun(validatorId);
  278:             emit ValidatorMarkedAsFrontRunned(_frontRunPubkey[i], validatorId);

   // other codes...
  285:         for (uint256 i; i < invalidSignatureValidatorsLength; ) {
  286:             uint256 validatorId = validatorIdByPubkey[_invalidSignaturePubkey[i]];
  287:             // only PRE_DEPOSIT status check will also include validatorId = 0 check
  288:             // as status for that will be INITIALIZED(default status)
  289:             onlyPreDepositValidator(validatorId);
  290:             validatorRegistry[validatorId].status = ValidatorStatus.INVALID_SIGNATURE;
  291:             emit ValidatorStatusMarkedAsInvalidSignature(_invalidSignaturePubkey[i], validatorId);



  311:     function withdrawnValidators(bytes[] calldata _pubkeys) external override {
  // other codes...
  317:         for (uint256 i; i < withdrawnValidatorCount; ) {
  // other codes...
  325:             emit ValidatorWithdrawn(_pubkeys[i], validatorId);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L95
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L175
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L278
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L291
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L325



```solidity
contracts\PermissionedPool.sol:

  129:     function fullDepositOnBeaconChain(bytes[] calldata _pubkey) external nonReentrant {
  // other codes...
  137:         for (uint256 i; i < pubkeyCount; ) {
  // other codes...
  162:             emit ValidatorDepositedOnBeaconChain(validatorId, _pubkey[i]);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L162


```solidity
contracts\PermissionlessNodeRegistry.sol:

  125:     function addValidatorKeys(
  // other codes...

  140:         for (uint256 i; i < keyCount; ) {
  // other codes...
  161:             emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId);


  199:         for (uint256 i; i < readyToDepositValidatorsLength; ) {
  // other codes...
  203:             emit ValidatorMarkedReadyToDeposit(_readyToDepositPubkey[i], validatorId);


  215:         for (uint256 i; i < frontRunValidatorsLength; ) {
  // other codes...
  219:             emit ValidatorMarkedAsFrontRunned(_frontRunPubkey[i], validatorId);

 
  225:         for (uint256 i; i < invalidSignatureValidatorsLength; ) {
  // other codes...
  229:             emit ValidatorStatusMarkedAsInvalidSignature(_invalidSignaturePubkey[i], validatorId);



  241:     function withdrawnValidators(bytes[] calldata _pubkeys) external override {
  // other codes...   
  247:         for (uint256 i; i < withdrawnValidatorCount; ) {
  // other codes...  
  255:             emit ValidatorWithdrawn(_pubkeys[i], validatorId);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L161
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L203
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L219
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L229
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L255


```solidity
contracts\PermissionlessPool.sol:

   85:     function preDepositOnBeaconChain(
  // other codes...  
   94:         for (uint256 i; i < pubkeyCount; ) {
  // other codes...  
  115:             emit ValidatorPreDepositedOnBeaconChain(_pubkey[i]);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L115


```solidity
contracts\PoolSelector.sol:
  119:     function updatePoolWeights(uint256[] calldata _poolTargets) external {
  // other codes... 
  130:         for (uint256 i = 0; i < poolTargetLength; i++) {
  131:             totalWeight += _poolTargets[i];
  132:             poolWeights[poolIdArray[i]] = _poolTargets[i];
  133:             emit UpdatedPoolWeight(poolIdArray[i], _poolTargets[i]);
  134:         }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L133


```solidity
contracts\PoolUtils.sol:

  72:     function processValidatorExitList(bytes[] calldata _pubkeys) external override {
  // other codes... 
  75:         for (uint256 i; i < exitValidatorCount; ) {
  76:             emit ExitValidator(_pubkeys[i]);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L76


```solidity
contracts\StaderStakePoolsManager.sol:

  215:     function depositETHOverTargetWeight() external override nonReentrant {
  // other codes... 
  232:         for (uint256 i = 0; i < poolCount; i++) {
  // other codes...
  244:             emit ETHTransferredToPool(i, poolAddress, validatorToDeposit * poolDepositSize);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L244



### [G-06] Use ``assembly`` to write _address storage values_ 

40 results - 18 files:

```solidity
contracts\factory\VaultFactory.sol:

  28:         staderConfig = IStaderConfig(_staderConfig);

  88:         staderConfig = IStaderConfig(_staderConfig);

  95:         vaultProxyImplementation = _vaultProxyImpl;

```
[VaultFactory.sol#L28](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L28), [VaultFactory.sol#L88](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L88), [VaultFactory.sol#L95](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L95)

```solidity
contracts\Auction.sol:

   37:         staderConfig = IStaderConfig(_staderConfig);

  140:         staderConfig = IStaderConfig(_staderConfig);

```
[Auction.sol#L37](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L37), [Auction.sol#L140](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L140)


```solidity
contracts\Penalty.sol:

  42:         staderConfig = IStaderConfig(_staderConfig);

  43:         ratedOracleAddress = _ratedOracleAddress;

  86:         ratedOracleAddress = _ratedOracleAddress;

  93:         staderConfig = IStaderConfig(_staderConfig);

```
[Penalty.sol#L42-L43](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L42-L43), [Penalty.sol#L86]( https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L86), [Penalty.sol#L93]( https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L93)


```solidity
contracts\ETHx.sol:

  37:         staderConfig = IStaderConfig(_staderConfig);

  87:         staderConfig = IStaderConfig(_staderConfig);

```
[ETHx.sol#L37](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L37), [ETHx.sol#L87](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L87)


```solidity
contracts\OperatorRewardsCollector.sol:

  34:         staderConfig = IStaderConfig(_staderConfig);

  58:         staderConfig = IStaderConfig(_staderConfig);

```
[OperatorRewardsCollector.sol#L34](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L34), [OperatorRewardsCollector.sol#L58](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L58), 


```solidity
contracts\PermissionedNodeRegistry.sol:

   72:         staderConfig = IStaderConfig(_staderConfig);

  461:         staderConfig = IStaderConfig(_staderConfig);

```
[PermissionedNodeRegistry.sol#L72](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L72), [PermissionedNodeRegistry.sol#L461](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L461)


```solidity
contracts\PermissionedPool.sol:

   47:         staderConfig = IStaderConfig(_staderConfig);

  250:         staderConfig = IStaderConfig(_staderConfig);

```
[PermissionedPool.sol#L47](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L47), [PermissionedPool.sol#L250](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L250)


```solidity
contracts\PermissionlessNodeRegistry.sol:

   72:         staderConfig = IStaderConfig(_staderConfig);

  356:         staderConfig = IStaderConfig(_staderConfig);

```
[PermissionlessNodeRegistry.sol#L72](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L72), [PermissionlessNodeRegistry.sol#L356](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L356)


```solidity
contracts\PermissionlessPool.sol:

   45:         staderConfig = IStaderConfig(_staderConfig);

  212:         staderConfig = IStaderConfig(_staderConfig);

```
[PermissionlessPool.sol#L45](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L45), [PermissionlessPool.sol#L212](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L212)


```solidity
contracts\PoolSelector.sol:

   41:         staderConfig = IStaderConfig(_staderConfig);

  149:         staderConfig = IStaderConfig(_staderConfig);

```
[PoolSelector.sol#L41](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L41), [PoolSelector.sol#L149](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L149), 


```solidity
contracts\PoolUtils.sol:

  29:         staderConfig = IStaderConfig(_staderConfig);

  86:         staderConfig = IStaderConfig(_staderConfig);

```
[PoolUtils.sol#L29](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L29), [PoolUtils.sol#L86](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L86)


```solidity
contracts\SDCollateral.sol:

   33:         staderConfig = IStaderConfig(_staderConfig);

  115:         staderConfig = IStaderConfig(_staderConfig);

```
[SDCollateral.sol#L33](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L33), [SDCollateral.sol#L115](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L115)


```solidity
contracts\SocializingPool.sol:

   47:         staderConfig = IStaderConfig(_staderConfig);

  182:         staderConfig = IStaderConfig(_staderConfig);

```
[SocializingPool.sol#L47](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L47), [SocializingPool.sol#L182](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L182)


```solidity
contracts\StaderInsuranceFund.sol:

  31:         staderConfig = IStaderConfig(_staderConfig);

  71:         staderConfig = IStaderConfig(_staderConfig);

```
[StaderInsuranceFund.sol#L31](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L31), [StaderInsuranceFund.sol#L71](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L71)


```solidity
contracts\StaderOracle.sol:

   75:         staderConfig = IStaderConfig(_staderConfig);

  511:         staderConfig = IStaderConfig(_staderConfig);

```
[StaderOracle.sol#L75](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L75), [StaderOracle.sol#L511](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L511)


```solidity
contracts\StaderStakePoolsManager.sol:

   58:         staderConfig = IStaderConfig(_staderConfig);

  118:         staderConfig = IStaderConfig(_staderConfig);

```
[StaderStakePoolsManager.sol#L58](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L58), [StaderStakePoolsManager.sol#L118](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L118)


```solidity
contracts\UserWithdrawalManager.sol:

  60:         staderConfig = IStaderConfig(_staderConfig);

  86:         staderConfig = IStaderConfig(_staderConfig);

```
[UserWithdrawalManager.sol#L60](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L60), [UserWithdrawalManager.sol#L86](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L86)


```solidity
contracts\VaultProxy.sol:

  34:         staderConfig = IStaderConfig(_staderConfig);

  59:         staderConfig = IStaderConfig(_staderConfig);

  70:         owner = _owner;

```
[VaultProxy.sol#L34](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L34), [VaultProxy.sol#L59](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L59), [VaultProxy.sol#L70](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L70)


**Recommendation Code:**
```diff

contracts\VaultProxy.sol:

- 70:         owner = _owner;
+                  assembly {                      
+                      sstore(owner .slot, _owner)
+                  }                               
          
```


### [G-07] Duplicated require()/if() checks should be refactored to a modifier or function

7 results - 3 files:

```solidity
contracts\PermissionedNodeRegistry.sol:

  590:         if (_pageNumber == 0) {
  591:             revert PageNumberIsZero();
  592:         }



  625:         if (_pageNumber == 0) {
  626:             revert PageNumberIsZero();
  627:         }

```
[PermissionedNodeRegistry.sol#L590-L592](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L590-L592), [PermissionedNodeRegistry.sol#L625-L627](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L625-L627)


```solidity
contracts\PermissionlessNodeRegistry.sol:

  496:         if (_pageNumber == 0) {
  497:             revert PageNumberIsZero();
  498:         }



  531:         if (_pageNumber == 0) {
  532:             revert PageNumberIsZero();
  533:         }



  565:         if (_pageNumber == 0) {
  566:             revert PageNumberIsZero();
  567:         }

```
[PermissionlessNodeRegistry.sol#L496-L498](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L496-L498), [PermissionlessNodeRegistry.sol#L531-L533](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L531-L533), [PermissionlessNodeRegistry.sol#L565-L567](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L565-L567)



```solidity
contracts\Auction.sol:
   80          LotItem storage lotItem = lots[lotId];
   81:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();


   97:         LotItem storage lotItem = lots[lotId];
   98:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();


  108          LotItem storage lotItem = lots[lotId];
  109:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();


  122          LotItem storage lotItem = lots[lotId];
  123:         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

```
[Auction.sol#L82](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L82), [Auction.sol#L98](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L98), [Auction.sol#L109](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L109), [Auction.sol#L123](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L123)



```diff
contracts\Auction.sol:

+      modifier checkEndBlock() {
+          LotItem storage lotItem = lots[lotId];
+          if (block.number <= lotItem.endBlock) revert AuctionNotEnded();

```


### [G-08] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.


3 results - 3 files:

```solidity
contracts\StaderOracle.sol:

  150:         if (
  151:             submissionCount == trustedNodesCount / 2 + 1 &&
  152:             _exchangeRate.reportingBlockNumber > exchangeRate.reportingBlockNumber
  153:         ) {
  154:             updateWithInLimitER(
  155:                 _exchangeRate.totalETHBalance,
  156:                 _exchangeRate.totalETHXSupply,
  157:                 _exchangeRate.reportingBlockNumber
  158:             );
  159:         }
  160      }



  189:         if (
  190:             !staderConfig.onlyManagerRole(msg.sender) &&
  191:             erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY > block.number
  192:         ) {
  193:             revert CooldownNotComplete();
  194:         }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L147-L157
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L189-L194


```solidity
contracts\StaderConfig.sol:

  513:         if (
  514:             !(variablesMap[MIN_DEPOSIT_AMOUNT] != 0 &&
  515:                 variablesMap[MIN_WITHDRAW_AMOUNT] != 0 &&
  516:                 variablesMap[MIN_DEPOSIT_AMOUNT] <= variablesMap[MAX_DEPOSIT_AMOUNT] &&
  517:                 variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MAX_WITHDRAW_AMOUNT] &&
  518:                 variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MIN_DEPOSIT_AMOUNT] &&
  519:                 variablesMap[MAX_WITHDRAW_AMOUNT] >= variablesMap[MAX_DEPOSIT_AMOUNT])
  520:         ) {
  521:             revert InvalidLimits();
  522:         }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L513-L523


```solidity
contracts\SocializingPool.sol:

  146:             if (_amountSD[i] == 0 && _amountETH[i] == 0) {
  147:                 revert InvalidAmount();
  148:             }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L146-L148


```solidity
contracts\ValidatorWithdrawalVault.sol:

  35:         if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
  36:             emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
  37:             revert InvalidRewardAmount();
  38:         }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L35



**Recomendation Code:**
```diff
contracts\ValidatorWithdrawalVault.sol:

- 35:         if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
+ 35:         if (!staderConfig.onlyOperatorRole(msg.sender)) {
+                   if (totalRewards > staderConfig.getRewardsThreshold()) {
  36:             emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
  37:             revert InvalidRewardAmount();
+                   }  
  38:         }

```


### [G-09] Using ``delete``  instead of setting mapping/struct ``0`` saves gas

2 results - 2 files:

```solidity
contracts\Auction.sol:

  113:         lotItem.sdAmount = 0;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L113


```solidity
contracts\Penalty.sol:

  146:         totalPenaltyAmount[pubkey] = 0;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L146


**Recommendation code:**
```diff
contracts\Penalty.sol:

- 146:         totalPenaltyAmount[pubkey] = 0;
+ 146:         delete totalPenaltyAmount[pubkey];

```

### [G-10] Do not calculate constants variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

1 result - 1 file:

```diff
contracts\StaderOracle.sol:

- 26:     uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days

+         // 7200 * 7 = 50_400;  (7 days)
+ 26:     uint256 public constant MAX_ER_UPDATE_FREQUENCY = 50_400;

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L26