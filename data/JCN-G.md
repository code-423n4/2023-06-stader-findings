# Summary
Some optimizations were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.16`, `optimizer on`, and `200 runs`. Optimizations that were not benchmarked are explained via EVM gas costs and opcodes. 

**Please note that a fair amount of functions that undergo optimizations are not benchmarked in the tests. Therefore, the total gas saved in the functions below may actually be greater when considering the gas saved from instances that don't include benchmarks**.

Below are the overall average gas savings for the following tested functions (with all the optimizations applied):

| Function |    Before   |    After   | Avg Gas Savings |
| ------ | -------- | -------- | ------- |
| Auction.claimSD |  8693  |  8641 |  52 | 
| Auction.createLot |  105780  |  103958 | 1822 | 
| NodeELRewardVault.withdraw |  122581  |  120166 | 2415 | 
| OperatorRewardsCollector.claim |  35757  |  35690 |  67 | 
| Penalty.updateTotalPenaltyAmount |  53839  |  51089 |  2750 | 
| PermissionedNodeRegistry.addValidatorKeys |  973836  |  962403 |  11433 |
| PermissionedNodeRegistry.withdrawnValidators |  30196  |  14323 |  15873 |
| PermissionlessNodeRegistry.addValidatorKeys |  839030  |  829233 |  9797 |
| PermissionlessNodeRegistry.markValidatorReadyToDeposit |  81811  |  81623 |  188 |
| PermissionlessNodeRegistry.withdrawnValidators |  35136  |  18608 |  16528 |
| PermissionlessNodeRegistry.updateDepositStatusAndBlock |  34034  |  14278 |  19756 |
| SDCollateral.withdraw |  21499  |  21169 |  330 |
| SDCollateral.slashValidatorSD |  98107  |  93733 |  4374 |
| ValidatorWithdrawalVault.distributeRewards |  85468  |  81847 |  3621 |
| ValidatorWithdrawalVault.settleFunds |  130849  |  117750 |  13099 |

**Total gas saved across all listed functions: 102105**

*Notes*: 

- The [Gas report](#gasreport-output-with-all-optimizations-applied) output, after all optimizations have been applied, can be found at the end of the report.
- The final diff for the contract, with all the optimizations applied, can be found [here](https://gist.github.com/0xJCN/40a25671cde00e756fcfc43b7a5ec4af).
- The `Avg` gas for `NodeELRewardVault.withdraw` changes between tests and so the `Max` column (which remains the same) is used when benchmarking this function.

## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:| 
| [G-01](#state-variables-can-be-cached-instead-of-re-reading-them-from-storage) | State variables can be cached instead of re-reading them from storage | 11 | 
| [G-02](#state-variables-can-be-packed-into-fewer-storage-slots) | State variables can be packed into fewer storage slots | 6 | 
| [G-03](#structs-can-be-packed-into-fewer-storage-slots) | Structs can be packed into fewer storage slots | 3 | 
| [G-04](#refactor-externalinternal-function-to-avoid-unnecessary-external-calls) | Refactor `external`/`internal` function to avoid unnecessary External Calls | 4 | 
| [G-05](#refactor-externalinternal-function-to-avoid-unnecessary-sload) | Refactor `external`/`internal` function to avoid unnecessary SLOAD | 8 | 
| [G-06](#cache-state-variables-outside-of-loop-to-avoid-readingwriting-storage-on-every-iteration) | Cache state variables outside of loop to avoid reading/writing storage on every iteration | 4 | 
| [G-07](#cache-external-calls-outside-of-loop-to-avoid-re-calling-function-on-each-iteration) | Cache external calls outside of loop to avoid re-calling function on each iteration | 2 | 
| [G-08](#refactor-code-to-avoid-unnecessary-sload) | Refactor code to avoid unnecessary SLOAD | 1 | 
| [G-09](#utilize-return-value-from-internal-function-to-avoid-unnecessary-sload) | Utilize return value from internal function to avoid unnecessary SLOAD | 2 | 
| [G-10](#multiple-address-mappings-can-be-combined-into-a-single-mapping-of-an-address-to-a-struct-where-appropriate) | Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate | 1 | 
| [G-11](#using-storage-instead-of-memory-for-structsarrays-saves-gas) | Using storage instead of memory for structs/arrays saves gas | 1 | 
| [G-12](#using-bools-for-storage-incurs-overhead) | Using bools for storage incurs overhead| - | 
| [G-13](#multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache) | Multiple accesses of a mapping/array should use a local variable cache | 4 | 
| [G-14](#combine-events-to-save-2-glogtopic-375-gas) | Combine events to save two `Glogtopic (375 gas)` | - | 
| [G-15](#call-environment-variables-directly-instead-of-caching-them) | Call environment variables directly instead of caching them | - | 
| [G-16](#use-assembly-to-perform-efficient-back-to-back-calls) | Use assembly to perform efficient back-to-back calls | 3 | 

## State variables can be cached instead of re-reading them from storage
Caching of a state variable replaces each `Gwarmaccess (100 gas)` with a much cheaper stack read.

**Note: These are instances missed by the Automated Report**.

Total Instances: `11`

Estimated Gas Saved: `11 * 100 = 1100`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L629-L630

### Cache `submissionCountKeys[_submissionCountKey]` to save 1 SLOAD
```solidity
File: contracts/StaderOracle.sol
629:        submissionCountKeys[_submissionCountKey]++; // @audit: 1st sload
630:        _submissionCount = submissionCountKeys[_submissionCountKey]; // @audit: 2nd sload
```
```diff
diff --git a/contracts/StaderOracle.sol b/contracts/StaderOracle.sol
index ba007c5..0dc382d 100644
--- a/contracts/StaderOracle.sol
+++ b/contracts/StaderOracle.sol
@@ -626,8 +626,9 @@ contract StaderOracle is IStaderOracle, AccessControlUpgradeable, PausableUpgrad
             revert DuplicateSubmissionFromNode();
         }
         nodeSubmissionKeys[_nodeSubmissionKey] = true;
-        submissionCountKeys[_submissionCountKey]++;
-        _submissionCount = submissionCountKeys[_submissionCountKey];
+        uint8 submissionCountKeyUpdated = submissionCountKeys[_submissionCountKey] + 1;
+        submissionCountKeys[_submissionCountKey] = submissionCountKeyUpdated;
+        _submissionCount = submissionCountKeyUpdated;
     }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L67-L82

### Cache `totalOperatorETHRewardsRemaining` and `totalOperatorSDRewardsRemaining` to save 2 SLOADs
```solidity
File: contracts/SocializingPool.sol
67:        if (
68:            _rewardsData.operatorETHRewards + _rewardsData.userETHRewards + _rewardsData.protocolETHRewards >
69:            address(this).balance - totalOperatorETHRewardsRemaining // @audit: 1st sload
70:        ) {
71:            revert InsufficientETHRewards();
72:        }
73:        if (
74:            _rewardsData.operatorSDRewards >
75:            IERC20(staderConfig.getStaderToken()).balanceOf(address(this)) - totalOperatorSDRewardsRemaining // @audit: 1st sload
76:        ) {
77:            revert InsufficientSDRewards();
78:        }
79:
80:        handledRewards[_rewardsData.index] = true;
81:        totalOperatorETHRewardsRemaining += _rewardsData.operatorETHRewards; // @audit: 2nd sload
82:        totalOperatorSDRewardsRemaining += _rewardsData.operatorSDRewards; // @audit: 2nd sload
```
```diff
diff --git a/contracts/SocializingPool.sol b/contracts/SocializingPool.sol
index 19d8cb2..9bdf9a9 100644
--- a/contracts/SocializingPool.sol
+++ b/contracts/SocializingPool.sol
@@ -64,22 +64,24 @@ contract SocializingPool is
         if (handledRewards[_rewardsData.index]) {
             revert RewardAlreadyHandled();
         }
+        uint256 _totalOperatorETHRewardsRemaining = totalOperatorSDRewardsRemaining;
         if (
             _rewardsData.operatorETHRewards + _rewardsData.userETHRewards + _rewardsData.protocolETHRewards >
-            address(this).balance - totalOperatorETHRewardsRemaining
+            address(this).balance - _totalOperatorETHRewardsRemaining
         ) {
             revert InsufficientETHRewards();
         }
+        uint256 _totalOperatorSDRewardsRemaining = totalOperatorSDRewardsRemaining;
         if (
             _rewardsData.operatorSDRewards >
-            IERC20(staderConfig.getStaderToken()).balanceOf(address(this)) - totalOperatorSDRewardsRemaining
+            IERC20(staderConfig.getStaderToken()).balanceOf(address(this)) - _totalOperatorSDRewardsRemaining
         ) {
             revert InsufficientSDRewards();
         }

         handledRewards[_rewardsData.index] = true;
-        totalOperatorETHRewardsRemaining += _rewardsData.operatorETHRewards;
-        totalOperatorSDRewardsRemaining += _rewardsData.operatorSDRewards;
+        totalOperatorETHRewardsRemaining = _totalOperatorETHRewardsRemaining + _rewardsData.operatorETHRewards;
+        totalOperatorSDRewardsRemaining = _totalOperatorSDRewardsRemaining + _rewardsData.operatorSDRewards;

         lastReportedRewardsData = _rewardsData;
         rewardsDataMap[_rewardsData.index] = _rewardsData;
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L91-L96

### Use already cached `sdBalance` to save 1 SLOAD
```solidity
File: contracts/SDCollateral.sol
91:        uint256 sdBalance = operatorSDBalance[_operator]; // @audit: 1st sload
92:        uint256 sdSlashed = Math.min(_sdToSlash, sdBalance);
93:        if (sdSlashed == 0) {
94:            return;
95:        }
96:        operatorSDBalance[_operator] -= sdSlashed; // @audit: 2nd sload
```
```diff
diff --git a/contracts/SDCollateral.sol b/contracts/SDCollateral.sol
index 0b5e22b..70f03df 100644
--- a/contracts/SDCollateral.sol
+++ b/contracts/SDCollateral.sol
@@ -93,7 +93,7 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
         if (sdSlashed == 0) {
             return;
         }
-        operatorSDBalance[_operator] -= sdSlashed;
+        operatorSDBalance[_operator] = sdBalance - sdSlashed;
         IAuction(staderConfig.getAuctionContract()).createLot(sdSlashed);
         emit SDSlashed(_operator, staderConfig.getAuctionContract(), sdSlashed);
     }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L60-L65

### Use already cached `opSDBalance` to save 1 SLOAD
```solidity
File: contracts/SDCollateral.sol
60:        uint256 opSDBalance = operatorSDBalance[operator]; // @audit: 1st sload
61:
62:        if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
63:            revert InsufficientSDToWithdraw(opSDBalance);
64:        }
65:        operatorSDBalance[operator] -= _requestedSD; // @audit: 2nd sload
```
```diff
diff --git a/contracts/SDCollateral.sol b/contracts/SDCollateral.sol
index 0b5e22b..292817a 100644
--- a/contracts/SDCollateral.sol
+++ b/contracts/SDCollateral.sol
@@ -62,7 +62,7 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
         if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
             revert InsufficientSDToWithdraw(opSDBalance);
         }
-        operatorSDBalance[operator] -= _requestedSD;
+        operatorSDBalance[operator] = opSDBalance - _requestedSD;

         // cannot use safeERC20 as this contract is an upgradeable contract
         if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L49-L58

### Cache `duration` and emit new struct values to save 3 SLOADs
```solidity
File: contracts/Auction.sol
49:        lots[nextLot].startBlock = block.number;
50:        lots[nextLot].endBlock = block.number + duration;
51:        lots[nextLot].sdAmount = _sdAmount;
52:
53:        LotItem storage lotItem = lots[nextLot];
54:
55:        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
56:            revert SDTransferFailed();
57:        }
58:        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement); // @audit: unneccesary SLOADs for `lotItem` variables
```
```diff
diff --git a/contracts/Auction.sol b/contracts/Auction.sol
index ae5fd02..ab6feeb 100644
--- a/contracts/Auction.sol
+++ b/contracts/Auction.sol
@@ -47,7 +47,8 @@ contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableU

     function createLot(uint256 _sdAmount) external override whenNotPaused {
         lots[nextLot].startBlock = block.number;
-        lots[nextLot].endBlock = block.number + duration;
+        uint256 _duration = duration;
+        lots[nextLot].endBlock = block.number + _duration;
         lots[nextLot].sdAmount = _sdAmount;

         LotItem storage lotItem = lots[nextLot];
@@ -55,7 +56,7 @@ contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableU
         if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
             revert SDTransferFailed();
         }
-        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
+        emit LotCreated(nextLot, _sdAmount, block.number, block.number + _duration, bidIncrement);
         nextLot++;
     }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L83-L90

### Cache `lotItem.highestBidder` and `lotItem.sdAmount` to save 3 SLOADs
```solidity
File: contracts/Auction.sol
83:        if (msg.sender != lotItem.highestBidder) revert notQualified(); // @audit: 1st sload
84:        if (lotItem.sdClaimed) revert AlreadyClaimed();
85:
86:        lotItem.sdClaimed = true;
87:        if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) { // @audit: 2nd sload & 1st sload
88:            revert SDTransferFailed();
89:        }
90:        emit SDClaimed(lotId, lotItem.highestBidder, lotItem.sdAmount); // @audit: 3rd sload & 2nd sload
```
```diff
diff --git a/contracts/Auction.sol b/contracts/Auction.sol
index ae5fd02..7322df5 100644
--- a/contracts/Auction.sol
+++ b/contracts/Auction.sol
@@ -80,14 +80,16 @@ contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableU
     function claimSD(uint256 lotId) external override {
         LotItem storage lotItem = lots[lotId];
         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
-        if (msg.sender != lotItem.highestBidder) revert notQualified();
+        address _highestBidder = lotItem.highestBidder;
+        if (msg.sender != _highestBidder) revert notQualified();
         if (lotItem.sdClaimed) revert AlreadyClaimed();

         lotItem.sdClaimed = true;
-        if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {
+        uint256 _sdAmount = lotItem.sdAmount;
+        if (!IERC20(staderConfig.getStaderToken()).transfer(_highestBidder, _sdAmount)) {
             revert SDTransferFailed();
         }
-        emit SDClaimed(lotId, lotItem.highestBidder, lotItem.sdAmount);
+        emit SDClaimed(lotId, _highestBidder, _sdAmount);
     }
```

## State variables can be packed into fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.

Total Instances: `6`

Estimated Gas Saved: `15 (slots) * 2000 = 30000`

**Note: These are instances missed by the Automated Report**.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L37-L39

### Pack `staderConfig`, `lastExcessETHDepositBlock`, and `excessETHDepositCoolDown` into one storage slot to save 2 SLOTs (~4000 gas)

**Note: In order to do this optimization we will need to reduce the `uint` types for `lastExcessETHDepositBlock` and `excessETHDepositCoolDown` to `uint48`. This `uint` type should be large enough to hold block numbers and time values.**
```solidity
File: contracts/StaderStakePoolsManager.sol
37:    IStaderConfig public staderConfig;
38:    uint256 public lastExcessETHDepositBlock;
39:    uint256 public excessETHDepositCoolDown;
```
```diff
diff --git a/contracts/StaderStakePoolsManager.sol b/contracts/StaderStakePoolsManager.sol
index c4bb7d5..9ebddd5 100644
--- a/contracts/StaderStakePoolsManager.sol
+++ b/contracts/StaderStakePoolsManager.sol
@@ -35,8 +35,8 @@ contract StaderStakePoolsManager is
     using Math for uint256;
     using SafeMath for uint256;
     IStaderConfig public staderConfig;
-    uint256 public lastExcessETHDepositBlock;
-    uint256 public excessETHDepositCoolDown;
+    uint48 public lastExcessETHDepositBlock;
+    uint48 public excessETHDepositCoolDown;

     /// @custom:oz-upgrades-unsafe-allow constructor
     constructor() {
@@ -53,7 +53,7 @@ contract StaderStakePoolsManager is
         __AccessControl_init();
         __Pausable_init();
         __ReentrancyGuard_init();
-        lastExcessETHDepositBlock = block.number;
+        lastExcessETHDepositBlock = uint48(block.number);
         excessETHDepositCoolDown = 3 * 7200;
         staderConfig = IStaderConfig(_staderConfig);
         _grantRole(DEFAULT_ADMIN_ROLE, _admin);
@@ -108,7 +108,7 @@ contract StaderStakePoolsManager is

     function updateExcessETHDepositCoolDown(uint256 _excessETHDepositCoolDown) external {
         UtilLib.onlyManagerRole(msg.sender, staderConfig);
-        excessETHDepositCoolDown = _excessETHDepositCoolDown;
+        excessETHDepositCoolDown = uint48(_excessETHDepositCoolDown);
         emit UpdatedExcessETHDepositCoolDown(_excessETHDepositCoolDown);
     }

@@ -238,7 +238,7 @@ contract StaderStakePoolsManager is
             uint256 poolDepositSize = staderConfig.getStakedEthPerNode() -
                 IPoolUtils(poolUtils).getCollateralETH(poolIdArray[i]);

-            lastExcessETHDepositBlock = block.number;
+            lastExcessETHDepositBlock = uint48(block.number);
             //slither-disable-next-line arbitrary-send-eth
             IStaderPoolBase(poolAddress).stakeUserETHToBeaconChain{value: validatorToDeposit * poolDepositSize}();
             emit ETHTransferredToPool(i, poolAddress, validatorToDeposit * poolDepositSize);
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L18-L41

### Pack `erInspectionMode`, `isPORFeedBasedERData`, `erInspectionModeStartBlock`, `staderConfig`, and `exchangeRate` into one slot to save 4 SLOTs (~8000 gas). 

### Pack `safeMode`, `reportingBlockNumberForWithdrawnValidators`, and `inspectionModeExchangeRate` to save an additional 2 SLOTs (~4000 gas). 

### Total SLOTs saved: 6 SLOTs (~12000 gas).

**Note: This storage space is flagged in the automated report. However, the suggested storage layout in the report savs 2 slots while the suggested layout above saves 6 slots. In order for this optimization to work we will need to reduce the `uint` value for state variables (and variables in the `ExchangeRate` struct) that hold block numbers. `uint` type `>=` `uint40` should be sufficient to hold block numbers**.
```solidity
File: contracts/StaderOracle.sol
18:    bool public override erInspectionMode;
19:    bool public override isPORFeedBasedERData;
20:    SDPriceData public lastReportedSDPriceData;
21:    IStaderConfig public override staderConfig;
22:    ExchangeRate public inspectionModeExchangeRate;
23:    ExchangeRate public exchangeRate;
24:    ValidatorStats public validatorStats;
25:
26:    uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days
27:    uint256 public constant ER_CHANGE_MAX_BPS = 10000;
28:    uint256 public override erChangeLimit;
29:    uint256 public constant MIN_TRUSTED_NODES = 5;
30:
31:    /// @inheritdoc IStaderOracle
32:    uint256 public override reportingBlockNumberForWithdrawnValidators;
33:    /// @inheritdoc IStaderOracle
34:    uint256 public override trustedNodesCount;
35:    /// @inheritdoc IStaderOracle
36:    uint256 public override lastReportedMAPDIndex;
37:    uint256 public override erInspectionModeStartBlock;
38:
39:    // indicate the health of protocol on beacon chain
40:    // enabled by `MANAGER` if heavy slashing on protocol on beacon chain
41:    bool public override safeMode;
```
```diff
diff --git a/contracts/StaderOracle.sol b/contracts/StaderOracle.sol
index ba007c5..1fa1138 100644
--- a/contracts/StaderOracle.sol
+++ b/contracts/StaderOracle.sol
@@ -17,10 +17,10 @@ import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.
 contract StaderOracle is IStaderOracle, AccessControlUpgradeable, PausableUpgradeable, ReentrancyGuardUpgradeable {
     bool public override erInspectionMode;
     bool public override isPORFeedBasedERData;
-    SDPriceData public lastReportedSDPriceData;
+    uint40 public override erInspectionModeStartBlock;
     IStaderConfig public override staderConfig;
-    ExchangeRate public inspectionModeExchangeRate;
     ExchangeRate public exchangeRate;
+    SDPriceData public lastReportedSDPriceData;
     ValidatorStats public validatorStats;

     uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days
@@ -29,16 +29,13 @@ contract StaderOracle is IStaderOracle, AccessControlUpgradeable, PausableUpgrad
     uint256 public constant MIN_TRUSTED_NODES = 5;

     /// @inheritdoc IStaderOracle
-    uint256 public override reportingBlockNumberForWithdrawnValidators;
+    bool public override safeMode;
+    uint208 public override reportingBlockNumberForWithdrawnValidators;
+    ExchangeRate public inspectionModeExchangeRate;
     /// @inheritdoc IStaderOracle
     uint256 public override trustedNodesCount;
     /// @inheritdoc IStaderOracle
     uint256 public override lastReportedMAPDIndex;
-    uint256 public override erInspectionModeStartBlock;
-
-    // indicate the health of protocol on beacon chain
-    // enabled by `MANAGER` if heavy slashing on protocol on beacon chain
-    bool public override safeMode;

     /// @inheritdoc IStaderOracle
     mapping(address => bool) public override isTrustedNode;
@@ -432,7 +429,7 @@ contract StaderOracle is IStaderOracle, AccessControlUpgradeable, PausableUpgrad
             submissionCount == trustedNodesCount / 2 + 1 &&
             _withdrawnValidators.reportingBlockNumber > reportingBlockNumberForWithdrawnValidators
         ) {
-            reportingBlockNumberForWithdrawnValidators = _withdrawnValidators.reportingBlockNumber;
+            reportingBlockNumberForWithdrawnValidators = uint208(_withdrawnValidators.reportingBlockNumber);
             INodeRegistry(_withdrawnValidators.nodeRegistry).withdrawnValidators(_withdrawnValidators.sortedPubkeys);

             // Emit withdrawn validators updated event
@@ -666,10 +663,10 @@ contract StaderOracle is IStaderOracle, AccessControlUpgradeable, PausableUpgrad
                 newExchangeRate <= ((currentExchangeRate * (ER_CHANGE_MAX_BPS + erChangeLimit)) / ER_CHANGE_MAX_BPS))
         ) {
             erInspectionMode = true;
-            erInspectionModeStartBlock = block.number;
+            erInspectionModeStartBlock = uint40(block.number);
             inspectionModeExchangeRate.totalETHBalance = _newTotalETHBalance;
             inspectionModeExchangeRate.totalETHXSupply = _newTotalETHXSupply;
-            inspectionModeExchangeRate.reportingBlockNumber = _reportingBlockNumber;
+            inspectionModeExchangeRate.reportingBlockNumber = uint40(_reportingBlockNumber);
             emit ERInspectionModeActivated(erInspectionMode, block.timestamp);
             return;
         }
@@ -683,7 +680,7 @@ contract StaderOracle is IStaderOracle, AccessControlUpgradeable, PausableUpgrad
     ) internal {
         exchangeRate.totalETHBalance = _totalETHBalance;
         exchangeRate.totalETHXSupply = _totalETHXSupply;
-        exchangeRate.reportingBlockNumber = _reportingBlockNumber;
+        exchangeRate.reportingBlockNumber = uint40(_reportingBlockNumber);

         // Emit balances updated event
         emit ExchangeRateUpdated(
```
```diff
diff --git a/contracts/interfaces/IStaderOracle.sol b/contracts/interfaces/IStaderOracle.sol
index d514c32..544a4ef 100644
--- a/contracts/interfaces/IStaderOracle.sol
+++ b/contracts/interfaces/IStaderOracle.sol
@@ -31,7 +31,7 @@ struct MissedAttestationReportInfo {
 /// @notice This struct holds data related to the exchange rate between ETH and ETHX.
 struct ExchangeRate {
     /// @notice The block number when the exchange rate was last updated.
-    uint256 reportingBlockNumber;
+    uint40 reportingBlockNumber;
     /// @notice The total balance of Ether (ETH) in the system.
     uint256 totalETHBalance;
     /// @notice The total supply of the liquid staking token (ETHX) in the system.
@@ -247,7 +247,7 @@ interface IStaderOracle {

     function erChangeLimit() external view returns (uint256);

-    function reportingBlockNumberForWithdrawnValidators() external view returns (uint256);
+    function reportingBlockNumberForWithdrawnValidators() external view returns (uint208);

     // returns the count of trusted nodes
     function trustedNodesCount() external view returns (uint256);
@@ -255,7 +255,7 @@ interface IStaderOracle {
     //returns the latest consensus index for missed attestation penalty data report
     function lastReportedMAPDIndex() external view returns (uint256);

-    function erInspectionModeStartBlock() external view returns (uint256);
+    function erInspectionModeStartBlock() external view returns (uint40);

     function safeMode() external view returns (bool);
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L24-L31

### Pack `staderConfig` and `lastReportedRewardsData` into together in storage to save 2 SLOTs (~4000 gas)

**Note: In order for this optimization to work we will need to efficiently pack the `RewardsData struct`. We can then declare `staderConfig (160 bits)` into storage after `lastReportedRewardsData` and this will pack `staderConfig` with `poolId (uint8)` and `reportingBlockNumber (uint48)`, saving 2 SLOTs**.
```solidity
File: contracts/SocializingPool.sol
24:    IStaderConfig public override staderConfig;
25:    uint256 public override totalOperatorETHRewardsRemaining;
26:    uint256 public override totalOperatorSDRewardsRemaining;
27:    uint256 public override initialBlock;
28:
29:    mapping(address => mapping(uint256 => bool)) public override claimedRewards;
30:    mapping(uint256 => bool) public handledRewards;
31:    RewardsData public lastReportedRewardsData;
```
```diff
diff --git a/contracts/SocializingPool.sol b/contracts/SocializingPool.sol
index 19d8cb2..9cf98dd 100644
--- a/./contracts/SocializingPool.sol
+++ b/./contracts/SocializingPoolNew.sol
@@ -21,6 +21,7 @@ contract SocializingPool is
     PausableUpgradeable,
     ReentrancyGuardUpgradeable
 {
+    RewardsData public lastReportedRewardsData;
     IStaderConfig public override staderConfig;
     uint256 public override totalOperatorETHRewardsRemaining;
     uint256 public override totalOperatorSDRewardsRemaining;
@@ -28,7 +29,6 @@ contract SocializingPool is

     mapping(address => mapping(uint256 => bool)) public override claimedRewards;
     mapping(uint256 => bool) public handledRewards;
-    RewardsData public lastReportedRewardsData;
     mapping(uint256 => RewardsData) public rewardsDataMap;

     /// @custom:oz-upgrades-unsafe-allow constructor
```
```diff
diff --git a/contracts/interfaces/ISocializingPool.sol b/contracts/interfaces/ISocializingPool.sol
index aceee8b..759724d 100644
--- a/contracts/interfaces/ISocializingPool.sol
+++ b/contracts/interfaces/ISocializingPool.sol
@@ -7,14 +7,10 @@ import './IStaderConfig.sol';
 /// @title RewardsData
 /// @notice This struct holds rewards merkleRoot and rewards split
 struct RewardsData {
-    /// @notice The block number when the rewards data was last updated
-    uint256 reportingBlockNumber;
     /// @notice The index of merkle tree or rewards cycle
     uint256 index;
     /// @notice The merkle root hash
     bytes32 merkleRoot;
-    /// @notice pool id of operators
-    uint8 poolId;
     /// @notice operator ETH rewards for index cycle
     uint256 operatorETHRewards;
     /// @notice user ETH rewards for index cycle
@@ -23,6 +19,10 @@ struct RewardsData {
     uint256 protocolETHRewards;
     /// @notice operator SD rewards for index cycle
     uint256 operatorSDRewards;
+    /// @notice pool id of operators
+    uint8 poolId;
+    /// @notice The block number when the rewards data was last updated
+    uint48 reportingBlockNumber;
 }

 interface ISocializingPool {
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L15-L16

### Pack `staderConfig` and `nextLot` into one storage slot to save 1 SLOT (~2000 gas)

**Note: We can also potentialy pack `staderConfig`, `nextLot`, and `duration` into one slot OR `staderConfig` + `nextLot` and `bidIncrement` + `duration` into 2 slots to save 2 SLOTs (~4000 gas). However, these storage arrangements result in failed test cases when `updateDuration` and `updateBidIncrement` are called with very large numbers (i.e. `type(uint128).max`). Since those two functions are privileged, precautions can be made to *not* update those variables with excessively large values.**

```solidity
File: contracts/Auction.sol
15:    IStaderConfig public override staderConfig;
16:    uint256 public override nextLot;
```
```diff
diff --git a/contracts/Auction.sol b/contracts/AuctionNew.sol
index ae5fd02..c9f9e37 100644
--- a/contracts/AuctionNew.sol
+++ b/contracts/AuctionNew.sol
@@ -13,7 +13,7 @@ import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.

 contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableUpgradeable, ReentrancyGuardUpgradeable {
     IStaderConfig public override staderConfig;
-    uint256 public override nextLot;
+    uint96 public override nextLot;
     uint256 public override bidIncrement;
     uint256 public override duration;
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L24-L29

### Pack `staderConfig`, `protocolFee`, and `operatorFee` into one storage slot to save 2 SLOTs (~4000 gas)

**Note: `protocolFee` and `operatorFee` is enforced to always be less than MAX_COMMISSION_LIMIT_BIPS. See [here](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L236)**.

```solidity
File: contracts/PermissionedPool.sol
24:    IStaderConfig public staderConfig;
25:    // @inheritdoc IStaderPoolBase
26:    uint256 public override protocolFee;
27:
28:    // @inheritdoc IStaderPoolBase
29:    uint256 public override operatorFee;
```
```diff
diff --git a/contracts/PermissionedPool.sol b/contracts/PermissionedPool.sol
index 85ec85d..1f831f9 100644
--- a/contracts/PermissionedPool.sol
+++ b/contracts/PermissionedPool.sol
@@ -23,10 +23,10 @@ contract PermissionedPool is IStaderPoolBase, Initializable, AccessControlUpgrad

     IStaderConfig public staderConfig;
     // @inheritdoc IStaderPoolBase
-    uint256 public override protocolFee;
+    uint48 public override protocolFee;

     // @inheritdoc IStaderPoolBase
-    uint256 public override operatorFee;
+    uint48 public override operatorFee;

     uint256 public preDepositValidatorCount;

@@ -238,8 +238,8 @@ contract PermissionedPool is IStaderPoolBase, Initializable, AccessControlUpgrad
         if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {
             revert InvalidCommission();
         }
-        protocolFee = _protocolFee;
-        operatorFee = _operatorFee;
+        protocolFee = uint48(_protocolFee);
+        operatorFee = uint48(_operatorFee);

         emit UpdatedCommissionFees(_protocolFee, _operatorFee);
     }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L21-L29

### Pack `staderConfig`, `protocolFee`, and `operatorFee` into one storage slot to save 2 SLOTs (~4000 gas)

**Note: `protocolFee` and `operatorFee` is enforced to always be less than MAX_COMMISSION_LIMIT_BIPS. See [here](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L66)**.

```solidity
File: contracts/PermissionlessPool
21:    IStaderConfig public staderConfig;
22:
23:    uint256 public constant DEPOSIT_NODE_BOND = 3 ether;
24:
25:    /// @inheritdoc IStaderPoolBase
26:    uint256 public override protocolFee;
27:
28:    /// @inheritdoc IStaderPoolBase
29:    uint256 public override operatorFee;
```
```diff
diff --git a/contracts/PermissionlessPool.sol b/contracts/PermissionlessPool.sol
index 046761d..25ba1dc 100644
--- a/contracts/PermissionlessPool.sol
+++ b/contracts/PermissionlessPool.sol
@@ -23,10 +23,10 @@ contract PermissionlessPool is IStaderPoolBase, Initializable, AccessControlUpgr
     uint256 public constant DEPOSIT_NODE_BOND = 3 ether;

     /// @inheritdoc IStaderPoolBase
-    uint256 public override protocolFee;
+    uint48 public override protocolFee;

     /// @inheritdoc IStaderPoolBase
-    uint256 public override operatorFee;
+    uint48 public override operatorFee;

     uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

@@ -68,8 +68,8 @@ contract PermissionlessPool is IStaderPoolBase, Initializable, AccessControlUpgr
         if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {
             revert InvalidCommission();
         }
-        protocolFee = _protocolFee;
-        operatorFee = _operatorFee;
+        protocolFee = uint48(_protocolFee);
+        operatorFee = uint48(_operatorFee);

         emit UpdatedCommissionFees(_protocolFee, _operatorFee);
     }
```

## Structs can be packed into fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs) we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.

Total Instances: `3`

Estimated Gas Saved: `4 (slots) * 2000 = 8000`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L41-L47

### Pack `owner` and `requestBlock` into one storage slot to save 1 SLOT (~2000 gas)

**Note: In order to do this optimization we will need to reduce the `uint` type for `requestBlock`. `uint48` should suffice for variables that hold block numbers.**
```solidity
File: contracts/UserWithdrawalManager.sol
41:    struct UserWithdrawInfo {
42:        address payable owner; // address that can claim eth on behalf of this request
43:        uint256 ethXAmount; //amount of ethX share locked for withdrawal
44:        uint256 ethExpected; //eth requested according to given share and exchangeRate
45:        uint256 ethFinalized; // final eth for claiming according to finalize exchange rate
46:        uint256 requestBlock; // block number of withdraw request
47:    }
```
```diff
diff --git a/contracts/UserWithdrawalManager.sol b/contracts/UserWithdrawalManager.sol
index f563434..e7954dd 100644
--- a/contracts/UserWithdrawalManager.sol
+++ b/contracts/UserWithdrawalManager.sol
@@ -40,10 +40,10 @@ contract UserWithdrawalManager is
     /// @notice structure representing a user request for withdrawal.
     struct UserWithdrawInfo {
         address payable owner; // address that can claim eth on behalf of this request
+        uint96 requestBlock; // block number of withdraw request
         uint256 ethXAmount; //amount of ethX share locked for withdrawal
         uint256 ethExpected; //eth requested according to given share and exchangeRate
         uint256 ethFinalized; // final eth for claiming according to finalize exchange rate
-        uint256 requestBlock; // block number of withdraw request
     }

     /// @custom:oz-upgrades-unsafe-allow constructor
@@ -103,7 +103,7 @@ contract UserWithdrawalManager is
         }
         IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);
         ethRequestedForWithdraw += assets;
-        userWithdrawRequests[nextRequestId] = UserWithdrawInfo(payable(_owner), _ethXAmount, assets, 0, block.number);
+        userWithdrawRequests[nextRequestId] = UserWithdrawInfo(payable(_owner), uint96(block.number), _ethXAmount, assets, 0);
         requestIdsByUserAddress[_owner].push(nextRequestId);
         emit WithdrawRequestReceived(msg.sender, _owner, nextRequestId, _ethXAmount, assets);
         nextRequestId++;
```
```diff
diff --git a/contracts/interfaces/IUserWithdrawalManager.sol b/contracts/interfaces/IUserWithdrawalManager.sol
index 82fc86b..bd5a574 100644
--- a/contracts/interfaces/IUserWithdrawalManager.sol
+++ b/contracts/interfaces/IUserWithdrawalManager.sol
@@ -52,10 +52,10 @@ interface IUserWithdrawalManager {
         view
         returns (
             address payable owner,
+            uint96 requestTime,
             uint256 ethXAmount,
             uint256 ethExpected,
-            uint256 ethFinalized,
-            uint256 requestTime
+            uint256 ethFinalized
         );
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/interfaces/ISocializingPool.sol#L9-L26

### Pack `poolId` and `reportingBlockNumber` into one storage slot to save 1 SLOT (~2000 gas)

**Note: We will need to reduce the `uint` size of `reportingBlockNumber` in order for this optimization to work. Since `reportingBlockNumber` holds block numbers a `uint` size of `uint48` should suffice**.

```solidity
File: contracts/interfaces/ISocializingPool
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
diff --git a/contracts/interfaces/ISocializingPool.sol b/contracts/interfaces/ISocializingPool.sol
index aceee8b..ab49093 100644
--- a/contracts/interfaces/ISocializingPool.sol
+++ b/contracts/interfaces/ISocializingPool.sol
@@ -7,14 +7,10 @@ import './IStaderConfig.sol';
 /// @title RewardsData
 /// @notice This struct holds rewards merkleRoot and rewards split
 struct RewardsData {
-    /// @notice The block number when the rewards data was last updated
-    uint256 reportingBlockNumber;
     /// @notice The index of merkle tree or rewards cycle
     uint256 index;
     /// @notice The merkle root hash
     bytes32 merkleRoot;
-    /// @notice pool id of operators
-    uint8 poolId;
     /// @notice operator ETH rewards for index cycle
     uint256 operatorETHRewards;
     /// @notice user ETH rewards for index cycle
@@ -23,6 +19,10 @@ struct RewardsData {
     uint256 protocolETHRewards;
     /// @notice operator SD rewards for index cycle
     uint256 operatorSDRewards;
+    /// @notice pool id of operators
+    uint8 poolId;
+    /// @notice The block number when the rewards data was last updated
+    uint48 reportingBlockNumber;
 }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/interfaces/INodeRegistry.sol#L6-L15

### Pack `withdrawVaultAddress`, `depositBlock`, and `withdrawnBlock` into one storage slot to save 2 SLOTs (~4000 gas)

**Note: We will need to reduce the `uint` size of `depositBlock` and `withdrawnBlock` in order for this optimization to work. Since boths of those values hold block numbers a `uint` size of `uint48` should suffice**.

```solidity
File: contracts/interfaces/INodeRegistry.sol
6: struct Validator {
7:     ValidatorStatus status; // status of validator
8:     bytes pubkey; //pubkey of the validator
9:     bytes preDepositSignature; //signature for 1 ETH deposit on beacon chain
10:    bytes depositSignature; //signature for 31 ETH deposit on beacon chain
11:    address withdrawVaultAddress; //withdrawal vault address of validator
12:    uint256 operatorId; // stader network assigned Id
13:    uint256 depositBlock; // block number of the 31ETH deposit
14:    uint256 withdrawnBlock; //block number when oracle report validator as withdrawn
15: }
```
```diff
diff --git a/contracts/interfaces/INodeRegistry.sol b/contracts/interfaces/INodeRegistry.sol
index 5360913..ab614a5 100644
--- a/contracts/interfaces/INodeRegistry.sol
+++ b/contracts/interfaces/INodeRegistry.sol
@@ -8,10 +8,10 @@ struct Validator {
     bytes pubkey; //pubkey of the validator
     bytes preDepositSignature; //signature for 1 ETH deposit on beacon chain
     bytes depositSignature; //signature for 31 ETH deposit on beacon chain
-    address withdrawVaultAddress; //withdrawal vault address of validator
     uint256 operatorId; // stader network assigned Id
-    uint256 depositBlock; // block number of the 31ETH deposit
-    uint256 withdrawnBlock; //block number when oracle report validator as withdrawn
+    address withdrawVaultAddress; //withdrawal vault address of validator
+    uint48 depositBlock; // block number of the 31ETH deposit
+    uint48 withdrawnBlock; //block number when oracle report validator as withdrawn
 }

 struct Operator {
@@ -72,10 +72,10 @@ interface INodeRegistry {
             bytes calldata pubkey,
             bytes calldata preDepositSignature,
             bytes calldata depositSignature,
-            address withdrawVaultAddress,
             uint256 operatorId,
-            uint256 depositTime,
-            uint256 withdrawnTime
+            address withdrawVaultAddress,
+            uint48 depositTime,
+            uint48 withdrawnTime
         );

     // returns the operator struct given operator Id
```
```diff
diff --git a/contracts/library/UtilLib.sol b/contracts/library/UtilLib.sol
index 75b2bc2..11e1fbc 100644
--- a/contracts/library/UtilLib.sol
+++ b/contracts/library/UtilLib.sol
@@ -53,7 +53,7 @@ library UtilLib {
         IStaderConfig _staderConfig
     ) internal view returns (bytes memory) {
         address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);
-        (, bytes memory pubkey, , , address withdrawVaultAddress, , , ) = INodeRegistry(nodeRegistry).validatorRegistry(
+        (, bytes memory pubkey, , , , address withdrawVaultAddress, , ) = INodeRegistry(nodeRegistry).validatorRegistry(
             _validatorId
         );
         if (_addr != withdrawVaultAddress) {
@@ -69,7 +69,7 @@ library UtilLib {
         IStaderConfig _staderConfig
     ) internal view returns (address) {
         address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);
-        (, , , , address withdrawVaultAddress, uint256 operatorId, , ) = INodeRegistry(nodeRegistry).validatorRegistry(
+        (, , , , uint256 operatorId, address withdrawVaultAddress, , ) = INodeRegistry(nodeRegistry).validatorRegistry(
             _validatorId
         );
         if (_addr != withdrawVaultAddress) {
@@ -86,7 +86,7 @@ library UtilLib {
         IStaderConfig _staderConfig
     ) internal view {
         address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);
-        (, , , , address withdrawVaultAddress, , , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);
+        (, , , , , address withdrawVaultAddress, , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);
         if (_addr != withdrawVaultAddress) {
             revert CallerNotWithdrawVault();
         }
@@ -98,7 +98,7 @@ library UtilLib {
         IStaderConfig _staderConfig
     ) internal view returns (address) {
         address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);
-        (, , , , , uint256 operatorId, , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);
+        (, , , , uint256 operatorId, , , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);
         (, , , , address operatorAddress) = INodeRegistry(nodeRegistry).operatorStructById(operatorId);

         return operatorAddress;
@@ -148,7 +148,7 @@ library UtilLib {
         uint8 poolId = IPoolUtils(_staderConfig.getPoolUtils()).getValidatorPoolId(_pubkey);
         address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(poolId);
         uint256 validatorId = INodeRegistry(nodeRegistry).validatorIdByPubkey(_pubkey);
-        (, , , , address withdrawVaultAddress, , , ) = INodeRegistry(nodeRegistry).validatorRegistry(validatorId);
+        (, , , , , address withdrawVaultAddress, , ) = INodeRegistry(nodeRegistry).validatorRegistry(validatorId);
         return IVaultProxy(withdrawVaultAddress).vaultSettleStatus();
     }
```
```diff
diff --git a/contracts/PermissionedNodeRegistry.sol b/contracts/PermissionedNodeRegistry.sol
index ca8aa81..911a645 100644
--- a/contracts/PermissionedNodeRegistry.sol
+++ b/contracts/PermissionedNodeRegistry.sol
@@ -165,8 +165,8 @@ contract PermissionedNodeRegistry is
                 _pubkey[i],
                 _preDepositSignature[i],
                 _depositSignature[i],
-                withdrawVault,
                 operatorId,
+                withdrawVault,
                 0,
                 0
             );
@@ -320,7 +320,7 @@ contract PermissionedNodeRegistry is
                 revert UNEXPECTED_STATUS();
             }
             validatorRegistry[validatorId].status = ValidatorStatus.WITHDRAWN;
-            validatorRegistry[validatorId].withdrawnBlock = block.number;
+            validatorRegistry[validatorId].withdrawnBlock = uint48(block.number);
             IValidatorWithdrawalVault(validatorRegistry[validatorId].withdrawVaultAddress).settleFunds();
             emit ValidatorWithdrawn(_pubkeys[i], validatorId);
             unchecked {
@@ -375,7 +375,7 @@ contract PermissionedNodeRegistry is
      */
     function updateDepositStatusAndBlock(uint256 _validatorId) external override {
         UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());
-        validatorRegistry[_validatorId].depositBlock = block.number;
+        validatorRegistry[_validatorId].depositBlock = uint48(block.number);
         markValidatorDeposited(_validatorId);
         emit UpdatedValidatorDepositBlock(_validatorId, block.number);
     }
```
```diff
diff --git a/contracts/PermissionedPool.sol b/contracts/PermissionedPool.sol
index 85ec85d..b6ed01a 100644
--- a/contracts/PermissionedPool.sol
+++ b/contracts/PermissionedPool.sol
@@ -137,7 +137,7 @@ contract PermissionedPool is IStaderPoolBase, Initializable, AccessControlUpgrad
         for (uint256 i; i < pubkeyCount; ) {
             IPermissionedNodeRegistry(nodeRegistryAddress).onlyPreDepositValidator(_pubkey[i]);
             uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdByPubkey(_pubkey[i]);
-            (, , , bytes memory depositSignature, address withdrawVaultAddress, , , ) = INodeRegistry(
+            (, , , bytes memory depositSignature, , address withdrawVaultAddress, , ) = INodeRegistry(
                 nodeRegistryAddress
             ).validatorRegistry(validatorId);
             bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(
@@ -291,7 +291,7 @@ contract PermissionedPool is IStaderPoolBase, Initializable, AccessControlUpgrad
         address _ethDepositContract,
         uint256 _validatorId
     ) internal {
-        (, bytes memory pubkey, bytes memory preDepositSignature, , address withdrawVaultAddress, , , ) = INodeRegistry(
+        (, bytes memory pubkey, bytes memory preDepositSignature, , , address withdrawVaultAddress, , ) = INodeRegistry(
             _nodeRegistryAddress
         ).validatorRegistry(_validatorId);
```
```diff
diff --git a/contracts/PermissionlessNodeRegistry.sol b/contracts/PermissionlessNodeRegistry.sol
index 9640556..ba72064 100644
--- a/contracts/PermissionlessNodeRegistry.sol
+++ b/contracts/PermissionlessNodeRegistry.sol
@@ -150,8 +150,8 @@ contract PermissionlessNodeRegistry is
                 _pubkey[i],
                 _preDepositSignature[i],
                 _depositSignature[i],
-                withdrawVault,
                 operatorId,
+                withdrawVault,
                 0,
                 0
             );
@@ -250,7 +250,7 @@ contract PermissionlessNodeRegistry is
                 revert UNEXPECTED_STATUS();
             }
             validatorRegistry[validatorId].status = ValidatorStatus.WITHDRAWN;
-            validatorRegistry[validatorId].withdrawnBlock = block.number;
+            validatorRegistry[validatorId].withdrawnBlock = uint48(block.number);
             IValidatorWithdrawalVault(validatorRegistry[validatorId].withdrawVaultAddress).settleFunds();
             emit ValidatorWithdrawn(_pubkeys[i], validatorId);
             unchecked {
@@ -278,7 +278,7 @@ contract PermissionlessNodeRegistry is
      */
     function updateDepositStatusAndBlock(uint256 _validatorId) external override {
         UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_POOL());
-        validatorRegistry[_validatorId].depositBlock = block.number;
+        validatorRegistry[_validatorId].depositBlock = uint48(block.number);
         markValidatorDeposited(_validatorId);
         emit UpdatedValidatorDepositBlock(_validatorId, block.number);
     }
```
```diff
diff --git a/contracts/PermissionlessPool.sol b/contracts/PermissionlessPool.sol
index 046761d..3ef6f99 100644
--- a/contracts/PermissionlessPool.sol
+++ b/contracts/PermissionlessPool.sol
@@ -245,7 +245,7 @@ contract PermissionlessPool is IStaderPoolBase, Initializable, AccessControlUpgr
         uint256 _validatorId,
         uint256 _DEPOSIT_SIZE
     ) internal {
-        (, bytes memory pubkey, , bytes memory depositSignature, address withdrawVaultAddress, , , ) = INodeRegistry(
+        (, bytes memory pubkey, , bytes memory depositSignature, , address withdrawVaultAddress, , ) = INodeRegistry(
             _nodeRegistryAddress
         ).validatorRegistry(_validatorId);
```

## Refactor `external`/`internal` function to avoid unnecessary External Calls
The functions below perform external calls that are previously performed in the functions that invoke them. We can refactor the `external`/`internal` functions so we could pass the cached external calls into them and avoid the extra external calls that would otherwise take place in the `internal` functions.

Total Instances: `4`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L32-L39C36

*Gas Savings for `NodeELRewardVault.withdraw`, obtained via protocol's tests: Avg 1253 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  81428   |  122581  |  69510  |    3     |
| After  |  79357   |  120895  |  68257  |    3     |

```solidity
File: contracts/NodeELRewardVault.sol
32:        (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils()) // @audit: 1st external call for "getPoolUtils()"
33:            .calculateRewardShare(poolId, totalRewards);
34:
35:        // Distribute rewards
36:        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{value: userShare}();
37:        // slither-disable-next-line arbitrary-send-eth
38:        UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare); 
39:        address operator = UtilLib.getOperatorAddressByOperatorId(poolId, operatorId, staderConfig); // @audit: 2nd external call for "getPoolUtils()"
```
```diff
diff --git a/contracts/NodeELRewardVault.sol b/contracts/NodeELRewardVault.sol
index b10417a..1d5cba1 100644
--- a/contracts/NodeELRewardVault.sol
+++ b/contracts/NodeELRewardVault.sol
@@ -29,18 +29,29 @@ contract NodeELRewardVault is INodeELRewardVault {
         if (totalRewards == 0) {
             revert NotEnoughRewardToWithdraw();
         }
-        (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
-            .calculateRewardShare(poolId, totalRewards);
+        IPoolUtils poolUtils = IPoolUtils(staderConfig.getPoolUtils());
+        (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = poolUtils.calculateRewardShare(poolId, totalRewards);

         // Distribute rewards
         IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{value: userShare}();
         // slither-disable-next-line arbitrary-send-eth
         UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
-        address operator = UtilLib.getOperatorAddressByOperatorId(poolId, operatorId, staderConfig);
+        address operator = getOperatorAddressByOperatorId(poolId, operatorId, poolUtils);
         IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
             operator
         );

         emit Withdrawal(protocolShare, operatorShare, userShare);
     }
+
+    function getOperatorAddressByOperatorId(
+        uint8 _poolId,
+        uint256 _operatorId,
+        IPoolUtils poolUtils
+    ) internal view returns (address) {
+        address nodeRegistry = poolUtils.getNodeRegistry(_poolId);
+        (, , , , address operatorAddress) = INodeRegistry(nodeRegistry).operatorStructById(_operatorId);
+        return operatorAddress;
+    }
 }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L42-L49

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L58-L80

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L136

*Gas Savings for `ValidatorWithdrawalVault.distributeRewards`, obtained via protocol's tests: Avg 885 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  83852   |  161928  |  85468  |    4     |
| After  |  82966   |  160162  |  84583  |    4     |

*Gas Savings for `ValidatorWithdrawalVault.settleFunds`, obtained via protocol's tests: Avg 1818 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  182196  |  196636  |  130849 |    3     |
| After  |  179470  |  193909  |  129031 |    3     |

```solidity
File: contracts/ValidatorWithdrawalVault.sol
42:        (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
43:            .calculateRewardShare(poolId, totalRewards);
44:
45:        // Distribute rewards
46:        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}();
47:        UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
48:        IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
49:            getOperatorAddress(poolId, validatorId, staderConfig)

58:        address nodeRegistry = IPoolUtils(staderConfig.getPoolUtils()).getNodeRegistry(poolId);
...
79:        IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
80:            getOperatorAddress(poolId, validatorId, staderConfig)

136:        return UtilLib.getOperatorAddressByValidatorId(_poolId, _validatorId, _staderConfig);
```
```diff
diff --git a/contracts/ValidatorWithdrawalVault.sol b/contracts/ValidatorWithdrawalVault.sol
index eb3de50..817cee9 100644
--- a/contracts/ValidatorWithdrawalVault.sol
+++ b/contracts/ValidatorWithdrawalVault.sol
@@ -1,7 +1,6 @@
 // SPDX-License-Identifier: MIT
 pragma solidity 0.8.16;

-import './library/UtilLib.sol';
 import './library/ValidatorStatus.sol';

 import './VaultProxy.sol';
@@ -39,14 +38,14 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
         if (totalRewards == 0) {
             revert NotEnoughRewardToDistribute();
         }
-        (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
-            .calculateRewardShare(poolId, totalRewards);
+        IPoolUtils poolUtils = IPoolUtils(staderConfig.getPoolUtils());
+        (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = poolUtils.calculateRewardShare(poolId, totalRewards);

         // Distribute rewards
         IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}();
         UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
         IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
-            getOperatorAddress(poolId, validatorId, staderConfig)
+            getOperatorAddress(validatorId, poolUtils.getNodeRegistry(poolId))
         );
         emit DistributedRewards(userShare, operatorShare, protocolShare);
     }
@@ -77,7 +76,7 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
         IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}();
         UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
         IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
-            getOperatorAddress(poolId, validatorId, staderConfig)
+            getOperatorAddress(validatorId, nodeRegistry)
         );
         emit SettledFunds(userShare, operatorShare, protocolShare);
     }
@@ -129,11 +128,12 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
     }

     function getOperatorAddress(
-        uint8 _poolId,
         uint256 _validatorId,
-        IStaderConfig _staderConfig
+        address nodeRegistry
     ) internal view returns (address) {
-        return UtilLib.getOperatorAddressByValidatorId(_poolId, _validatorId, _staderConfig);
+        (, , , , , uint256 operatorId, , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);
+        (, , , , address operatorAddress) = INodeRegistry(nodeRegistry).operatorStructById(operatorId);
+        return operatorAddress;
     }

     function getUpdatedPenaltyAmount(
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L58-L76

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L139-L150

*Gas Savings for `ValidatorWithdrawalVault.settleFunds`, obtained via protocol's tests: Avg 3980 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  182196  |  196636  |  130849 |    3     |
| After  |  176227  |  190667  |  126869 |    3     |

```solidity
File: contracts/ValidatorWithdrawalVault.sol
58:        address nodeRegistry = IPoolUtils(staderConfig.getPoolUtils()).getNodeRegistry(poolId); // @audit: 1st set of external calls to get `nodeRegistry`
59:        if (msg.sender != nodeRegistry) {
60:            revert CallerNotNodeRegistryContract();
61:        }
62:        (uint256 userSharePrelim, uint256 operatorShare, uint256 protocolShare) = calculateValidatorWithdrawalShare();
63:
64:        uint256 penaltyAmount = getUpdatedPenaltyAmount(poolId, validatorId, staderConfig); // @audit: additional 4 external calls
65:
66:        if (operatorShare < penaltyAmount) {
67:            ISDCollateral(staderConfig.getSDCollateral()).slashValidatorSD(validatorId, poolId);
68:            penaltyAmount = operatorShare;
69:        }
70:
71:        uint256 userShare = userSharePrelim + penaltyAmount;
72:        operatorShare = operatorShare - penaltyAmount;
73:
74:        // Final settlement
75:        vaultSettleStatus = true;
76:        IPenalty(staderConfig.getPenaltyContract()).markValidatorSettled(poolId, validatorId); // @audit: 1 external call for "getPenaltyContract()"

139:    function getUpdatedPenaltyAmount(
140:        uint8 _poolId,
141:        uint256 _validatorId,
142:        IStaderConfig _staderConfig
143:    ) internal returns (uint256) {
144:        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId); // @audit: 2nd set of external calls to get `nodeRegistry`
145:        (, bytes memory pubkey, , , , , , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);
146:        bytes[] memory pubkeyArray = new bytes[](1);
147:        pubkeyArray[0] = pubkey;
148:        IPenalty(_staderConfig.getPenaltyContract()).updateTotalPenaltyAmount(pubkeyArray); // @audit: 2nd external call for "getPenaltyContract()"
149:        return IPenalty(_staderConfig.getPenaltyContract()).totalPenaltyAmount(pubkey); // @audit: 3rd external call for "getPenaltyContract()"
150:    }
```
```diff
diff --git a/contracts/ValidatorWithdrawalVault.sol b/contracts/ValidatorWithdrawalVault.sol
index eb3de50..562f76b 100644
--- a/contracts/ValidatorWithdrawalVault.sol
+++ b/contracts/ValidatorWithdrawalVault.sol
@@ -60,8 +60,9 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
             revert CallerNotNodeRegistryContract();
         }
         (uint256 userSharePrelim, uint256 operatorShare, uint256 protocolShare) = calculateValidatorWithdrawalShare();
-
-        uint256 penaltyAmount = getUpdatedPenaltyAmount(poolId, validatorId, staderConfig);
+
+        IPenalty penalty = IPenalty(staderConfig.getPenaltyContract());
+        uint256 penaltyAmount = getUpdatedPenaltyAmount(validatorId, nodeRegistry, penalty);

         if (operatorShare < penaltyAmount) {
             ISDCollateral(staderConfig.getSDCollateral()).slashValidatorSD(validatorId, poolId);
@@ -73,7 +74,7 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {

         // Final settlement
         vaultSettleStatus = true;
-        IPenalty(staderConfig.getPenaltyContract()).markValidatorSettled(poolId, validatorId);
+        penalty.markValidatorSettled(poolId, validatorId);
         IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}();
         UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
         IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
@@ -137,15 +138,15 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
     }

     function getUpdatedPenaltyAmount(
-        uint8 _poolId,
         uint256 _validatorId,
-        IStaderConfig _staderConfig
+        address nodeRegistry,
+        IPenalty penalty
     ) internal returns (uint256) {
-        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);
         (, bytes memory pubkey, , , , , , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);
         bytes[] memory pubkeyArray = new bytes[](1);
         pubkeyArray[0] = pubkey;
-        IPenalty(_staderConfig.getPenaltyContract()).updateTotalPenaltyAmount(pubkeyArray);
-        return IPenalty(_staderConfig.getPenaltyContract()).totalPenaltyAmount(pubkey);
+        penalty.updateTotalPenaltyAmount(pubkeyArray);
+        return penalty.totalPenaltyAmount(pubkey);
     }
 }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L54-L62

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L85-L97

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L127-L129

*Gas Savings for `ValidatorWithdrawalVault.settleFunds`, obtained via protocol's tests: Avg 3311 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  182196  |  196636  |  130849 |    3     |
| After  |  177226  |  191666  |  127538 |    3     |

```solidity
File: contracts/ValidatorWithdrawalVault.sol
54:    function settleFunds() external override {
55:        uint8 poolId = VaultProxy(payable(address(this))).poolId(); // @audit: 1st external call
56:        uint256 validatorId = VaultProxy(payable(address(this))).id();
57:        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig(); // @audit: 1st external call
58:        address nodeRegistry = IPoolUtils(staderConfig.getPoolUtils()).getNodeRegistry(poolId); // @audit: 1st external call for `getPoolUtils()`
59:        if (msg.sender != nodeRegistry) {
60:            revert CallerNotNodeRegistryContract();
61:        }
62:        (uint256 userSharePrelim, uint256 operatorShare, uint256 protocolShare) = calculateValidatorWithdrawalShare(); // @audit: 2nd external calls

85:    function calculateValidatorWithdrawalShare()
86:        public
87:        view
88:        returns (
89:            uint256 _userShare,
90:            uint256 _operatorShare,
91:            uint256 _protocolShare
92:        )
93:    {
94:        uint8 poolId = VaultProxy(payable(address(this))).poolId(); // @audit: 2nd external call
95:        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig(); // @audit: 2nd external call
96:        uint256 TOTAL_STAKED_ETH = staderConfig.getStakedEthPerNode();
97:        uint256 collateralETH = getCollateralETH(poolId, staderConfig); // 0, incase of permissioned NOs // @audit: 2nd external call for `getPoolUtils()`

127:    function getCollateralETH(uint8 _poolId, IStaderConfig _staderConfig) internal view returns (uint256) {
128:        return IPoolUtils(_staderConfig.getPoolUtils()).getCollateralETH(_poolId); // @audit: 2nd external call for `getPoolUtils()`
129:    }
```
```diff
diff --git a/contracts/ValidatorWithdrawalVault.sol b/contracts/ValidatorWithdrawalVault.sol
index eb3de50..4b956e3 100644
--- a/contracts/ValidatorWithdrawalVault.sol
+++ b/contracts/ValidatorWithdrawalVault.sol
@@ -55,11 +55,12 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
         uint8 poolId = VaultProxy(payable(address(this))).poolId();
         uint256 validatorId = VaultProxy(payable(address(this))).id();
         IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
-        address nodeRegistry = IPoolUtils(staderConfig.getPoolUtils()).getNodeRegistry(poolId);
+        IPoolUtils poolUtils = IPoolUtils(staderConfig.getPoolUtils());
+        address nodeRegistry = poolUtils.getNodeRegistry(poolId);
         if (msg.sender != nodeRegistry) {
             revert CallerNotNodeRegistryContract();
         }
-        (uint256 userSharePrelim, uint256 operatorShare, uint256 protocolShare) = calculateValidatorWithdrawalShare();
+        (uint256 userSharePrelim, uint256 operatorShare, uint256 protocolShare) = _calculateValidatorWithdrawalShare(staderConfig, poolId, poolUtils);

         uint256 penaltyAmount = getUpdatedPenaltyAmount(poolId, validatorId, staderConfig);

@@ -91,10 +92,25 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
             uint256 _protocolShare
         )
     {
-        uint8 poolId = VaultProxy(payable(address(this))).poolId();
         IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
+        return _calculateValidatorWithdrawalShare(
+            staderConfig,
+            VaultProxy(payable(address(this))).poolId(),
+            IPoolUtils(staderConfig.getPoolUtils())
+        );
+    }
+
+    function _calculateValidatorWithdrawalShare(IStaderConfig staderConfig, uint8 poolId, IPoolUtils poolUtils)
+        internal
+        view
+        returns (
+            uint256 _userShare,
+            uint256 _operatorShare,
+            uint256 _protocolShare
+        )
+    {
         uint256 TOTAL_STAKED_ETH = staderConfig.getStakedEthPerNode();
-        uint256 collateralETH = getCollateralETH(poolId, staderConfig); // 0, incase of permissioned NOs
+        uint256 collateralETH = getCollateralETH(poolId, poolUtils); // 0, incase of permissioned NOs
         uint256 usersETH = TOTAL_STAKED_ETH - collateralETH;
         uint256 contractBalance = address(this).balance;

@@ -113,9 +129,7 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
             _userShare = usersETH;
         }
         if (totalRewards > 0) {
-            (uint256 userReward, uint256 operatorReward, uint256 protocolReward) = IPoolUtils(
-                staderConfig.getPoolUtils()
-            ).calculateRewardShare(poolId, totalRewards);
+            (uint256 userReward, uint256 operatorReward, uint256 protocolReward) = poolUtils.calculateRewardShare(poolId, totalRewards);
             _userShare += userReward;
             _operatorShare += operatorReward;
             _protocolShare += protocolReward;
@@ -124,8 +138,8 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {

     // HELPER METHODS

-    function getCollateralETH(uint8 _poolId, IStaderConfig _staderConfig) internal view returns (uint256) {
-        return IPoolUtils(_staderConfig.getPoolUtils()).getCollateralETH(_poolId);
+    function getCollateralETH(uint8 _poolId, IPoolUtils poolUtils) internal view returns (uint256) {
+        return poolUtils.getCollateralETH(_poolId);
     }

     function getOperatorAddress(
```

## Refactor `external`/`internal` function to avoid unnecessary SLOAD
The functions below read storage slots that are previously read in the functions that invoke them. We can refactor the `external`/`internal` functions so we could pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the `internal` functions.

Total Instances: `8`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L78-L84

*Gas Savings for `SDCollateral.slashValidatorSD`, obtained via protocol's tests: Avg 295 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  76118   |  176106  |  98107  |    6     |
| After  |  75884   |  175537  |  97812  |    6     |

```solidity
File: contracts/SDCollateral.sol
78:    function slashValidatorSD(uint256 _validatorId, uint8 _poolId) external override nonReentrant {
79:        address operator = UtilLib.getOperatorForValidSender(_poolId, _validatorId, msg.sender, staderConfig); // @audit: 1st sload
80:        isPoolThresholdValid(_poolId);
81:        PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];
82:        uint256 sdToSlash = convertETHToSD(poolThreshold.minThreshold); // @audit: 2nd & 3rd sloads
83:        slashSD(operator, sdToSlash); // @audit: 4th & 5th sloads
84:    }
```
```diff
diff --git a/contracts/SDCollateral.sol b/contracts/SDCollateral.sol
index 0b5e22b..04db8db 100644
--- a/contracts/SDCollateral.sol
+++ b/contracts/SDCollateral.sol
@@ -76,26 +76,27 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
     /// @dev callable only by respective withdrawVaults
     /// @param _validatorId validator SD collateral to slash
     function slashValidatorSD(uint256 _validatorId, uint8 _poolId) external override nonReentrant {
-        address operator = UtilLib.getOperatorForValidSender(_poolId, _validatorId, msg.sender, staderConfig);
+        IStaderConfig _staderConfig = staderConfig;
+        address operator = UtilLib.getOperatorForValidSender(_poolId, _validatorId, msg.sender, _staderConfig);
         isPoolThresholdValid(_poolId);
         PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];
-        uint256 sdToSlash = convertETHToSD(poolThreshold.minThreshold);
-        slashSD(operator, sdToSlash);
+        uint256 sdToSlash = _convertETHToSD(_staderConfig, poolThreshold.minThreshold);
+        slashSD(_staderConfig, operator, sdToSlash);
     }

     /// @notice used to slash operator SD, incase of operator default
     /// @dev do provide SD approval to auction contract using `maxApproveSD()`
     /// @param _operator which operator SD collateral to slash
     /// @param _sdToSlash amount of SD to slash
-    function slashSD(address _operator, uint256 _sdToSlash) internal {
+    function slashSD(IStaderConfig _staderConfig, address _operator, uint256 _sdToSlash) internal {
         uint256 sdBalance = operatorSDBalance[_operator];
         uint256 sdSlashed = Math.min(_sdToSlash, sdBalance);
         if (sdSlashed == 0) {
             return;
         }
         operatorSDBalance[_operator] -= sdSlashed;
-        IAuction(staderConfig.getAuctionContract()).createLot(sdSlashed);
-        emit SDSlashed(_operator, staderConfig.getAuctionContract(), sdSlashed);
+        IAuction(_staderConfig.getAuctionContract()).createLot(sdSlashed);
+        emit SDSlashed(_operator, _staderConfig.getAuctionContract(), sdSlashed);
     }

     /// @notice for max approval to auction contract for spending SD tokens
@@ -208,8 +209,12 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
     }

     function convertETHToSD(uint256 _ethAmount) public view override returns (uint256) {
-        uint256 sdPriceInETH = IStaderOracle(staderConfig.getStaderOracle()).getSDPriceInETH();
-        return (_ethAmount * staderConfig.getDecimals()) / sdPriceInETH;
+        return _convertETHToSD(staderConfig, _ethAmount);
+    }
+
+    function _convertETHToSD(IStaderConfig _staderConfig, uint256 _ethAmount) internal view returns (uint256) {
+        uint256 sdPriceInETH = IStaderOracle(_staderConfig.getStaderOracle()).getSDPriceInETH();
+        return (_ethAmount * _staderConfig.getDecimals()) / sdPriceInETH;
     }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L62-L68

*Gas Savings for `SDCollateral.withdraw`, obtained via protocol's tests: Avg 252 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  19651   |  33104   |  21499  |    3     |
| After  |  19220   |  33085   |  21247  |    3     |

```solidity
File: contracts/SDCollateral.sol
62:        if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) { // @audit: multiple sloads for `staderConfig`
63:            revert InsufficientSDToWithdraw(opSDBalance);
64:        }
65:        operatorSDBalance[operator] -= _requestedSD;
66:
67:        // cannot use safeERC20 as this contract is an upgradeable contract
68:        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) { // @audit: sload for `staderConfig`
```
```diff
diff --git a/contracts/SDCollateral.sol b/contracts/SDCollateral.sol
index 0b5e22b..f2e12db 100644
--- a/contracts/SDCollateral.sol
+++ b/contracts/SDCollateral.sol
@@ -58,14 +58,15 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
     function withdraw(uint256 _requestedSD) external override {
         address operator = msg.sender;
         uint256 opSDBalance = operatorSDBalance[operator];
-
-        if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
+
+        IStaderConfig _staderConfig = staderConfig;
+        if (opSDBalance < _getOperatorWithdrawThreshold(_staderConfig, operator) + _requestedSD) {
             revert InsufficientSDToWithdraw(opSDBalance);
         }
         operatorSDBalance[operator] -= _requestedSD;

         // cannot use safeERC20 as this contract is an upgradeable contract
-        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
+        if (!IERC20(_staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
             revert SDTransferFailed();
         }

@@ -142,10 +143,14 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,

     // returns sum of withdraw threshold accounting for all its(op's) validators
     function getOperatorWithdrawThreshold(address _operator) public view returns (uint256 operatorWithdrawThreshold) {
-        (uint8 poolId, , uint256 validatorCount) = getOperatorInfo(_operator);
+        return _getOperatorWithdrawThreshold(staderConfig, _operator);
+    }
+
+    function _getOperatorWithdrawThreshold(IStaderConfig _staderConfig, address _operator) internal view returns (uint256 operatorWithdrawThreshold) {
+        (uint8 poolId, , uint256 validatorCount) = getOperatorInfo(_staderConfig, _operator);
         isPoolThresholdValid(poolId);
         PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[poolId];
-        return convertETHToSD(poolThreshold.withdrawThreshold * validatorCount);
+        return _convertETHToSD(_staderConfig, poolThreshold.withdrawThreshold * validatorCount);
     }

     /// @notice checks if operator has enough SD collateral to onboard validators in a specific pool
@@ -191,7 +196,7 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
     }

     function getRewardEligibleSD(address _operator) external view override returns (uint256 _rewardEligibleSD) {
-        (uint8 poolId, , uint256 validatorCount) = getOperatorInfo(_operator);
+        (uint8 poolId, , uint256 validatorCount) = getOperatorInfo(staderConfig, _operator);

         isPoolThresholdValid(poolId);
         PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[poolId];
@@ -208,13 +213,17 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
     }

     function convertETHToSD(uint256 _ethAmount) public view override returns (uint256) {
-        uint256 sdPriceInETH = IStaderOracle(staderConfig.getStaderOracle()).getSDPriceInETH();
-        return (_ethAmount * staderConfig.getDecimals()) / sdPriceInETH;
+        return _convertETHToSD(staderConfig, _ethAmount);
+    }
+
+    function _convertETHToSD(IStaderConfig _staderConfig, uint256 _ethAmount) internal view returns (uint256) {
+        uint256 sdPriceInETH = IStaderOracle(_staderConfig.getStaderOracle()).getSDPriceInETH();
+        return (_ethAmount * _staderConfig.getDecimals()) / sdPriceInETH;
     }

     // HELPER

-    function getOperatorInfo(address _operator)
+    function getOperatorInfo(IStaderConfig _staderConfig, address _operator)
         internal
         view
         returns (
@@ -223,7 +232,7 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
             uint256 _validatorCount
         )
     {
-        IPoolUtils poolUtils = IPoolUtils(staderConfig.getPoolUtils());
+        IPoolUtils poolUtils = IPoolUtils(_staderConfig.getPoolUtils());
         _poolId = poolUtils.getOperatorPoolId(_operator);
         INodeRegistry nodeRegistry = INodeRegistry(poolUtils.getNodeRegistry(_poolId));
         _operatorId = nodeRegistry.operatorIDByAddress(_operator);
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L100-L107

*Gas Savings for `Penalty.updateTotalPenaltyAmount`, obtained via protocol's tests: Avg 79 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  51152   |  88271   |  53839  |    3     |
| After  |  51155   |  88151   |  53760  |    3     |

```solidity
File: contracts/Penalty.sol
100:        for (uint256 i; i < reportedValidatorCount; ) {
101:            if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) { // @audit: 1st sload
102:                revert ValidatorSettled();
103:            }
104:            bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
105:            // Retrieve the penalty for changing the fee recipient address based on Rated.network data.
106:            uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
107:            uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot); // @audit: 2nd sload
```
```diff
diff --git a/contracts/Penalty.sol b/contracts/Penalty.sol
index d6a28dc..9e0bfa6 100644
--- a/contracts/Penalty.sol
+++ b/contracts/Penalty.sol
@@ -98,13 +98,14 @@ contract Penalty is IPenalty, Initializable, AccessControlUpgradeable, Reentranc
     function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
         uint256 reportedValidatorCount = _pubkey.length;
         for (uint256 i; i < reportedValidatorCount; ) {
-            if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
+            IStaderConfig _staderConfig = staderConfig;
+            if (UtilLib.getValidatorSettleStatus(_pubkey[i], _staderConfig)) {
                 revert ValidatorSettled();
             }
             bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
             // Retrieve the penalty for changing the fee recipient address based on Rated.network data.
             uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
-            uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);
+            uint256 _missedAttestationPenalty = _calculateMissedAttestationPenalty(_staderConfig, pubkeyRoot);

             // Compute the total penalty for the given validator public key,
             // taking into account additional penalties and penalty reversals from the DAO.
@@ -119,6 +120,12 @@ contract Penalty is IPenalty, Initializable, AccessControlUpgradeable, Reentranc
         }
     }

+    function _calculateMissedAttestationPenalty(IStaderConfig _staderConfig, bytes32 _pubkeyRoot) internal view returns (uint256) {
+        return
+            IStaderOracle(_staderConfig.getStaderOracle()).missedAttestationPenalty(_pubkeyRoot) *
+            missedAttestationPenaltyPerStrike;
+    }
+
     /// @inheritdoc IPenalty
     function calculateMEVTheftPenalty(bytes32 _pubkeyRoot) public override returns (uint256) {
         // Retrieve the epochs in which the validator violated the fee recipient change rule.
@@ -130,9 +137,7 @@ contract Penalty is IPenalty, Initializable, AccessControlUpgradeable, Reentranc

     /// @inheritdoc IPenalty
     function calculateMissedAttestationPenalty(bytes32 _pubkeyRoot) public view override returns (uint256) {
-        return
-            IStaderOracle(staderConfig.getStaderOracle()).missedAttestationPenalty(_pubkeyRoot) *
-            missedAttestationPenaltyPerStrike;
+        return _calculateMissedAttestationPenalty(staderConfig, _pubkeyRoot);
     }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L146-L153

*Gas Savings for `PermissionedNodeRegistry.addValidatorKeys`, obtained via protocol's tests: Avg 140 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  1461878 |  1461878 |  973836 |    15    |
| After  |  1461630 |  1461630 |  973696 |    15    |

**Note: We can also use the cached state variable in line 154 to save a 2nd SLOAD. However, since line 154 is included in the automated report, it will not be included here**.

```solidity
File: contracts/PermissionedNodeRegistry.sol
146:        (uint256 keyCount, uint256 operatorTotalKeys) = checkInputKeysCountAndCollateral( // @audit: 1st sload for `staderConfig`
147:            _pubkey.length,
148:            _preDepositSignature.length,
149:            _depositSignature.length,
150:            operatorId
151:        );
152:
153:        address vaultFactory = staderConfig.getVaultFactory(); // @audit: 2nd sload
```
```diff
diff --git a/contracts/PermissionedNodeRegistry.sol b/contracts/PermissionedNodeRegistry.sol
index ca8aa81..f2e0fa2 100644
--- a/contracts/PermissionedNodeRegistry.sol
+++ b/contracts/PermissionedNodeRegistry.sol
@@ -143,14 +143,16 @@ contract PermissionedNodeRegistry is
         bytes[] calldata _depositSignature
     ) external override whenNotPaused {
         uint256 operatorId = onlyActiveOperator(msg.sender);
+        IStaderConfig _staderConfig = staderConfig;
         (uint256 keyCount, uint256 operatorTotalKeys) = checkInputKeysCountAndCollateral(
+            _staderConfig,
             _pubkey.length,
             _preDepositSignature.length,
             _depositSignature.length,
             operatorId
         );

-        address vaultFactory = staderConfig.getVaultFactory();
+        address vaultFactory = _staderConfig.getVaultFactory();
         address poolUtils = staderConfig.getPoolUtils();
         for (uint256 i; i < keyCount; ) {
             IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);
@@ -685,6 +687,7 @@ contract PermissionedNodeRegistry is

     // validate the input of `addValidatorKeys` function
     function checkInputKeysCountAndCollateral(
+        IStaderConfig _staderConfig,
         uint256 _pubkeyLength,
         uint256 _preDepositSignatureLength,
         uint256 _depositSignatureLength,
@@ -706,7 +709,7 @@ contract PermissionedNodeRegistry is
         //checks if operator has enough SD collateral for adding `keyCount` keys
         //SD threshold for permissioned NOs is 0 for phase1
         if (
-            !ISDCollateral(staderConfig.getSDCollateral()).hasEnoughSDCollateral(
+            !ISDCollateral(_staderConfig.getSDCollateral()).hasEnoughSDCollateral(
                 msg.sender,
                 POOL_ID,
                 totalNonTerminalKeys + keyCount
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L118-L129

### Pass a cached `nextOperatorId` variable into `onboardOperator` to save 1 SLOAD

**Note: Multiple other SLOADs for `nextOperatorId` are performed in `onboardOperator`. However, since those instances are included in the automated report they will not be included here**.
```solidity
File: contracts/PermissionedNodeRegistry.sol
118:        if (nextOperatorId > maxOperatorId) { // @audit: 1st sload
119:            revert MaxOperatorLimitReached();
120:        }
121:        if (!permissionList[msg.sender]) {
122:            revert NotAPermissionedNodeOperator();
123:        }
124:        //checks if operator already onboarded in any pool of protocol
125:        if (IPoolUtils(poolUtils).isExistingOperator(msg.sender)) {
126:            revert OperatorAlreadyOnBoardedInProtocol();
127:        }
128:        feeRecipientAddress = staderConfig.getPermissionedSocializingPool();
129:        onboardOperator(_operatorName, _operatorRewardAddress); // @audit: multiple sloads for `nextOperatorId`
```
```diff
diff --git a/contracts/PermissionedNodeRegistry.sol b/contracts/PermissionedNodeRegistry.sol
index ca8aa81..bea75a2 100644
--- a/contracts/PermissionedNodeRegistry.sol
+++ b/contracts/PermissionedNodeRegistry.sol
@@ -115,7 +115,8 @@ contract PermissionedNodeRegistry is
         }
         IPoolUtils(poolUtils).onlyValidName(_operatorName);
         UtilLib.checkNonZeroAddress(_operatorRewardAddress);
-        if (nextOperatorId > maxOperatorId) {
+        uint256 _nextOperatorId = nextOperatorId;
+        if (_nextOperatorId > maxOperatorId) {
             revert MaxOperatorLimitReached();
         }
         if (!permissionList[msg.sender]) {
@@ -126,7 +127,7 @@ contract PermissionedNodeRegistry is
             revert OperatorAlreadyOnBoardedInProtocol();
         }
         feeRecipientAddress = staderConfig.getPermissionedSocializingPool();
-        onboardOperator(_operatorName, _operatorRewardAddress);
+        onboardOperator(_nextOperatorId, _operatorName, _operatorRewardAddress);
         return feeRecipientAddress;
     }

@@ -658,8 +659,8 @@ contract PermissionedNodeRegistry is
         onlyPreDepositValidator(validatorId);
     }

-    function onboardOperator(string calldata _operatorName, address payable _operatorRewardAddress) internal {
-        operatorStructById[nextOperatorId] = Operator(true, true, _operatorName, _operatorRewardAddress, msg.sender);
+    function onboardOperator(uint256 _nextOperatorId, string calldata _operatorName, address payable _operatorRewardAddress) internal {
+        operatorStructById[_nextOperatorId] = Operator(true, true, _operatorName, _operatorRewardAddress, msg.sender);
         operatorIDByAddress[msg.sender] = nextOperatorId;
         socializingPoolStateChangeBlock[nextOperatorId] = block.number;
         nextOperatorId++;
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L100-L115

### Pass cached `staderConfig` variable into `preDepositOnBeaconChain` to save an SLOAD on every iteration

**Note: This optimization can be combined with the instances found by the automated report in order to also cache all occurances of `staderConfig` in this function**.

```solidity
File: contracts/PermissionedPool.sol
100:        for (uint256 i = 1; i < selectedOperatorCapacityLength; ) {
101:            uint256 validatorToDeposit = selectedOperatorCapacity[i];
102:            if (validatorToDeposit == 0) {
103:                continue;
104:            }
105:            increasePreDepositValidatorCount(validatorToDeposit);
106:            uint256 nextQueuedValidatorIndex = IPermissionedNodeRegistry(nodeRegistryAddress)
107:                .nextQueuedValidatorIndexByOperatorId(i);
108:
109:            for (
110:                uint256 index = nextQueuedValidatorIndex;
111:                index < nextQueuedValidatorIndex + validatorToDeposit;
112:                index++
113:            ) {
114:                uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdsByOperatorId(i, index); 
115:                preDepositOnBeaconChain(nodeRegistryAddress, vaultFactory, ethDepositContract, validatorId); // internal function reads `staderConfig`
```
```diff
diff --git a/contracts/PermissionedPool.sol b/contracts/PermissionedPool.sol
index 85ec85d..f06f357 100644
--- a/contracts/PermissionedPool.sol
+++ b/contracts/PermissionedPool.sol
@@ -97,6 +97,7 @@ contract PermissionedPool is IStaderPoolBase, Initializable, AccessControlUpgrad

         // i is the operator Id
         uint256 selectedOperatorCapacityLength = selectedOperatorCapacity.length;
+        IStaderConfig _staderConfig = staderConfig;
         for (uint256 i = 1; i < selectedOperatorCapacityLength; ) {
             uint256 validatorToDeposit = selectedOperatorCapacity[i];
             if (validatorToDeposit == 0) {
@@ -112,7 +113,7 @@ contract PermissionedPool is IStaderPoolBase, Initializable, AccessControlUpgrad
                 index++
             ) {
                 uint256 validatorId = INodeRegistry(nodeRegistryAddress).validatorIdsByOperatorId(i, index);
-                preDepositOnBeaconChain(nodeRegistryAddress, vaultFactory, ethDepositContract, validatorId);
+                preDepositOnBeaconChain(_staderConfig, nodeRegistryAddress, vaultFactory, ethDepositContract, validatorId);
             }
             IPermissionedNodeRegistry(nodeRegistryAddress).updateQueuedValidatorIndex(
                 i,
@@ -286,6 +287,7 @@ contract PermissionedPool is IStaderPoolBase, Initializable, AccessControlUpgrad

     // deposit `PRE_DEPOSIT_SIZE` for validator
     function preDepositOnBeaconChain(
+        IStaderConfig _staderConfig,
         address _nodeRegistryAddress,
         address _vaultFactory,
         address _ethDepositContract,
@@ -298,7 +300,7 @@ contract PermissionedPool is IStaderPoolBase, Initializable, AccessControlUpgrad
         bytes memory withdrawCredential = IVaultFactory(_vaultFactory).getValidatorWithdrawCredential(
             withdrawVaultAddress
         );
-        uint256 preDepositSize = staderConfig.getPreDepositSize();
+        uint256 preDepositSize = _staderConfig.getPreDepositSize();
         bytes32 depositDataRoot = this.computeDepositDataRoot(
             pubkey,
             preDepositSignature,
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L110-L114

### Pass a cached `nextOperatorId` variable into `onboardOperator` to save 1 SLOAD

**Note: Multiple other SLOADs for `nextOperatorId` are performed in `onboardOperator`. However, since those instances are included in the automated report they will not be included here**.

```solidity
File: contracts/PermissionlessNodeRegistry.sol
110:        nodeELRewardVaultByOperatorId[nextOperatorId] = nodeELRewardVault; // @audit: 1st sload
111:        feeRecipientAddress = _optInForSocializingPool
112:            ? staderConfig.getPermissionlessSocializingPool()
113:            : nodeELRewardVault; 
114:        onboardOperator(_optInForSocializingPool, _operatorName, _operatorRewardAddress); // @audit: multiple sloads for `nextOperatorId`
```
```diff
diff --git a/contracts/PermissionlessNodeRegistry.sol b/contracts/PermissionlessNodeRegistry.sol
index 9640556..704b977 100644
--- a/contracts/PermissionlessNodeRegistry.sol
+++ b/contracts/PermissionlessNodeRegistry.sol
@@ -107,11 +107,12 @@ contract PermissionlessNodeRegistry is
             POOL_ID,
             nextOperatorId
         );
-        nodeELRewardVaultByOperatorId[nextOperatorId] = nodeELRewardVault;
+        uint256 _nextOperatorId = nextOperatorId;
+        nodeELRewardVaultByOperatorId[_nextOperatorId] = nodeELRewardVault;
         feeRecipientAddress = _optInForSocializingPool
             ? staderConfig.getPermissionlessSocializingPool()
             : nodeELRewardVault;
-        onboardOperator(_optInForSocializingPool, _operatorName, _operatorRewardAddress);
+        onboardOperator(_nextOperatorId, _optInForSocializingPool, _operatorName, _operatorRewardAddress);
         return feeRecipientAddress;
     }

@@ -596,11 +597,12 @@ contract PermissionlessNodeRegistry is
     }

     function onboardOperator(
+        uint256 _nextOperatorId,
         bool _optInForSocializingPool,
         string calldata _operatorName,
         address payable _operatorRewardAddress
     ) internal {
-        operatorStructById[nextOperatorId] = Operator(
+        operatorStructById[_nextOperatorId] = Operator(
             true,
             _optInForSocializingPool,
             _operatorName,
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L131-L138

*Gas Savings for `PermissionlessNodeRegistry.addValidatorKeys`, obtained via protocol's tests: Avg 105 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  1461673 |  1461673 |  839030 |    14    |
| After  |  1461425 |  1461425 |  838925 |    14    |

**Note: We can also use the cached state variable in line 139 to save a 2nd SLOAD. However, since line 139 is included in the automated report, it will not be included here**.

```solidity
File: contracts/PermissionlessNodeRegistry.sol
131:        (uint256 keyCount, uint256 operatorTotalKeys) = checkInputKeysCountAndCollateral( // @audit: 1st sload for `staderConfig`
132:            _pubkey.length,
133:            _preDepositSignature.length,
134:            _depositSignature.length,
135:            operatorId
136:        );
137:
138:        address vaultFactory = staderConfig.getVaultFactory(); // @audit: 2nd sload
```
```diff
diff --git a/contracts/PermissionlessNodeRegistry.sol b/contracts/PermissionlessNodeRegistry.sol
index 9640556..beb410c 100644
--- a/contracts/PermissionlessNodeRegistry.sol
+++ b/contracts/PermissionlessNodeRegistry.sol
@@ -128,14 +128,16 @@ contract PermissionlessNodeRegistry is
         bytes[] calldata _depositSignature
     ) external payable override nonReentrant whenNotPaused {
         uint256 operatorId = onlyActiveOperator(msg.sender);
+        IStaderConfig _staderConfig = staderConfig;
         (uint256 keyCount, uint256 operatorTotalKeys) = checkInputKeysCountAndCollateral(
+            _staderConfig,
             _pubkey.length,
             _preDepositSignature.length,
             _depositSignature.length,
             operatorId
         );

-        address vaultFactory = staderConfig.getVaultFactory();
+        address vaultFactory = _staderConfig.getVaultFactory();
         address poolUtils = staderConfig.getPoolUtils();
         for (uint256 i; i < keyCount; ) {
             IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);
@@ -641,6 +643,7 @@ contract PermissionlessNodeRegistry is

     // validate the input of `addValidatorKeys` function
     function checkInputKeysCountAndCollateral(
+        IStaderConfig _staderConfig,
         uint256 _pubkeyLength,
         uint256 _preDepositSignatureLength,
         uint256 _depositSignatureLength,
@@ -666,7 +669,7 @@ contract PermissionlessNodeRegistry is
         }
         //checks if operator has enough SD collateral for adding `keyCount` keys
         if (
-            !ISDCollateral(staderConfig.getSDCollateral()).hasEnoughSDCollateral(
+            !ISDCollateral(_staderConfig.getSDCollateral()).hasEnoughSDCollateral(
                 msg.sender,
                 POOL_ID,
                 totalNonTerminalKeys + keyCount
```

## Cache state variables outside of loop to avoid reading/writing storage on every iteration
Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a `Gwarmaccess (100 gas)` per loop iteration.

Total Instances: `4`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L155-L180

*Gas Savings for `PermissionedNodeRegistry.addValidatorKeys`, obtained via protocol's tests: Avg 1297 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  1461878 |  1461878 |  973836 |    15    |
| After  |  1459933 |  1459933 |  972539 |    15    |

**Note: This optimization saves multiple SLOADs and an SSTORE per iteration. Caching all of the `nextValidatorId` variables inside of the for loop is caught in the automated report. However, the report doesn't specify if the cached variable should be declared outside or inside of the loop. If declared inside of the loop this would still result in at least one SLOAD per iteration. Here I am specifying declaring the variable outside of the for loop so that we only incur one SLOAD despite the number of iterations. Additionally, the automated report does not include any refactoring needed to avoid incurring an SSTORE per iteration**.

```solidity
File: contracts/PermissionedNodeRegistry.sol
155:        for (uint256 i; i < keyCount; ) {
156:            IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);
157:            address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(
158:                POOL_ID,
159:                operatorId,
160:                operatorTotalKeys + i, //operator totalKeys
161:                nextValidatorId // @audit: 1st sload + on every interation
162:            );
163:            validatorRegistry[nextValidatorId] = Validator( // @audit: 2nd sload + on every iteration
164:                ValidatorStatus.INITIALIZED,
165:                _pubkey[i],
166:                _preDepositSignature[i],
167:                _depositSignature[i],
168:                withdrawVault,
169:                operatorId,
170:                0,
171:                0
172:            );
173:            validatorIdByPubkey[_pubkey[i]] = nextValidatorId; // @audit: 3rd sload + on every iteration
174:            validatorIdsByOperatorId[operatorId].push(nextValidatorId); // @audit: 4th sload + on every iteration
175:            emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId); // @audit: 5th sload + on every iteration
176:            nextValidatorId++; // @audit: 6th sload and sstore on every iteration
177:            unchecked {
178:                ++i;
179:            }
180:        }
```
```diff
diff --git a/contracts/PermissionedNodeRegistry.sol b/contracts/PermissionedNodeRegistry.sol
index ca8aa81..c092aac 100644
--- a/contracts/PermissionedNodeRegistry.sol
+++ b/contracts/PermissionedNodeRegistry.sol
@@ -152,15 +152,16 @@ contract PermissionedNodeRegistry is

         address vaultFactory = staderConfig.getVaultFactory();
         address poolUtils = staderConfig.getPoolUtils();
+        uint256 _nextValidatorId = nextValidatorId;
         for (uint256 i; i < keyCount; ) {
             IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);
             address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(
                 POOL_ID,
                 operatorId,
                 operatorTotalKeys + i, //operator totalKeys
-                nextValidatorId
+                _nextValidatorId
             );
-            validatorRegistry[nextValidatorId] = Validator(
+            validatorRegistry[_nextValidatorId] = Validator(
                 ValidatorStatus.INITIALIZED,
                 _pubkey[i],
                 _preDepositSignature[i],
@@ -170,14 +171,15 @@ contract PermissionedNodeRegistry is
                 0,
                 0
             );
-            validatorIdByPubkey[_pubkey[i]] = nextValidatorId;
-            validatorIdsByOperatorId[operatorId].push(nextValidatorId);
-            emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId);
-            nextValidatorId++;
+            validatorIdByPubkey[_pubkey[i]] = _nextValidatorId;
+            validatorIdsByOperatorId[operatorId].push(_nextValidatorId);
+            emit AddedValidatorKey(msg.sender, _pubkey[i], _nextValidatorId);
+            _nextValidatorId++;
             unchecked {
                 ++i;
             }
         }
+        nextValidatorId = _nextValidatorId;
     }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L140-L166

*Gas Savings for `PermissionlessNodeRegistry.addValidatorKeys`, obtained via protocol's tests: Avg 1104 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  1461673 |  1461673 |  839030 |    14    |
| After  |  1459741 |  1459741 |  837926 |    14    |

**Note: This optimization saves multiple SLOADs and an SSTORE per iteration. Caching all of the `nextValidatorId` variables inside of the for loop is caught in the automated report. However, the report doesn't specify if the cached variable should be declared outside or inside of the loop. If declared inside of the loop this would still result in at least one SLOAD per iteration. Here I am specifying declaring the variable outside of the for loop so that we only incur one SLOAD despite the number of iterations. Additionally, the automated report does not include any refactoring needed to avoid incurring an SSTORE per iteration**.

```solidity
File: contracts/PermissionlessNodeRegistry.sol
140:        for (uint256 i; i < keyCount; ) {
141:            IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);
142:            address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(
143:                POOL_ID,
144:                operatorId,
145:                operatorTotalKeys + i, //operator totalKeys
146:                nextValidatorId // @audit: 1st sload + on every iteration
147:            );
148:            validatorRegistry[nextValidatorId] = Validator( // @audit: 2nd sload + on every iteration
149:                ValidatorStatus.INITIALIZED,
150:                _pubkey[i],
151:                _preDepositSignature[i],
152:                _depositSignature[i],
153:                withdrawVault,
154:                operatorId,
155:                0,
156:                0
157:            );
158:
159:            validatorIdByPubkey[_pubkey[i]] = nextValidatorId; // @audit: 3rd sload + on every iteration;
160:            validatorIdsByOperatorId[operatorId].push(nextValidatorId); // @audit: 4th sload + on every iteration;
161:            emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId); // @audit: 5th sload + on every iteration
162:            nextValidatorId++; // @audit: 6th sload and sstore on every iteration
163:            unchecked {
164:                ++i;
165:            }
166:        }
```
```diff
diff --git a/contracts/PermissionlessNodeRegistry.sol b/contracts/PermissionlessNodeRegistry.sol
index 9640556..56911cf 100644
--- a/contracts/PermissionlessNodeRegistry.sol
+++ b/contracts/PermissionlessNodeRegistry.sol
@@ -137,15 +137,16 @@ contract PermissionlessNodeRegistry is

         address vaultFactory = staderConfig.getVaultFactory();
         address poolUtils = staderConfig.getPoolUtils();
+        uint256 _nextValidatorId = nextValidatorId;
         for (uint256 i; i < keyCount; ) {
             IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);
             address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(
                 POOL_ID,
                 operatorId,
                 operatorTotalKeys + i, //operator totalKeys
-                nextValidatorId
+                _nextValidatorId
             );
-            validatorRegistry[nextValidatorId] = Validator(
+            validatorRegistry[_nextValidatorId] = Validator(
                 ValidatorStatus.INITIALIZED,
                 _pubkey[i],
                 _preDepositSignature[i],
@@ -156,14 +157,15 @@ contract PermissionlessNodeRegistry is
                 0
             );

-            validatorIdByPubkey[_pubkey[i]] = nextValidatorId;
-            validatorIdsByOperatorId[operatorId].push(nextValidatorId);
-            emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId);
-            nextValidatorId++;
+            validatorIdByPubkey[_pubkey[i]] = _nextValidatorId;
+            validatorIdsByOperatorId[operatorId].push(_nextValidatorId);
+            emit AddedValidatorKey(msg.sender, _pubkey[i], _nextValidatorId);
+            _nextValidatorId++;
             unchecked {
                 ++i;
             }
         }
+        nextValidatorId = _nextValidatorId;

         //slither-disable-next-line arbitrary-send-eth
         IPermissionlessPool(staderConfig.getPermissionlessPool()).preDepositOnBeaconChain{
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L199-L207

*Gas Savings for `PermissionlessNodeRegistry.markValidatorReadyToDeposit`, obtained via protocol's tests: Avg 189 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  76381   |  168703  |  81811  |    5     |
| After  |  76257   |  167903  |  81622  |    5     |

**Note: Here we are moving the logic from the `markKeyReadyToDeposit` internal function into the for loop and refactoring the code in order to avoid performing an SSTORE for `validatorQueueSize` on each iteration*.

```solidity
File: contracts/PermissionlessNodeRegistry.sol
199:        for (uint256 i; i < readyToDepositValidatorsLength; ) {
200:            uint256 validatorId = validatorIdByPubkey[_readyToDepositPubkey[i]];
201:            onlyInitializedValidator(validatorId);
202:            markKeyReadyToDeposit(validatorId); // @audit: 2 sloads + sstore on every iteration for `validatorQueueSize`
203:            emit ValidatorMarkedReadyToDeposit(_readyToDepositPubkey[i], validatorId);
204:            unchecked {
205:                ++i;
206:            }
207:        }
```
```diff
diff --git a/contracts/PermissionlessNodeRegistry.sol b/contracts/PermissionlessNodeRegistry.sol
index 9640556..70dfc68 100644
--- a/contracts/PermissionlessNodeRegistry.sol
+++ b/contracts/PermissionlessNodeRegistry.sol
@@ -195,16 +195,20 @@ contract PermissionlessNodeRegistry is
         ) {
             revert TooManyVerifiedKeysReported();
         }
-
+
+        uint256 _validatorQueueSize = validatorQueueSize;
         for (uint256 i; i < readyToDepositValidatorsLength; ) {
             uint256 validatorId = validatorIdByPubkey[_readyToDepositPubkey[i]];
             onlyInitializedValidator(validatorId);
-            markKeyReadyToDeposit(validatorId);
+            validatorRegistry[validatorId].status = ValidatorStatus.PRE_DEPOSIT;
+            queuedValidators[_validatorQueueSize] = validatorId;
+            _validatorQueueSize++;
             emit ValidatorMarkedReadyToDeposit(_readyToDepositPubkey[i], validatorId);
             unchecked {
                 ++i;
             }
         }
+        validatorQueueSize = _validatorQueueSize;

         if (frontRunValidatorsLength > 0) {
             IStaderInsuranceFund(staderConfig.getStaderInsuranceFund()).depositFund{
@@ -614,13 +618,6 @@ contract PermissionlessNodeRegistry is
         emit OnboardedOperator(msg.sender, _operatorRewardAddress, nextOperatorId - 1, _optInForSocializingPool);
     }

-    // mark validator  `PRE_DEPOSIT` after successful key verification and front run check
-    function markKeyReadyToDeposit(uint256 _validatorId) internal {
-        validatorRegistry[_validatorId].status = ValidatorStatus.PRE_DEPOSIT;
-        queuedValidators[validatorQueueSize] = _validatorId;
-        validatorQueueSize++;
-    }
-
     // handle front run validator by changing their status, deactivating operator and imposing penalty
     function handleFrontRun(uint256 _validatorId) internal {
         validatorRegistry[_validatorId].status = ValidatorStatus.FRONT_RUN;
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L132-L145

### Use stack variable to keep track of eth to withdraw in loop and update the `ethRequestedForWithdraw` outside of loop if we had at least one complete iteration. This will save `1 SSTORE + 1 SLOAD` per complete loop iteration.

**Note: This function is not benchmarked in the protocol's tests and therefore the exact gas savings are not included here.**.
```solidity
File: contracts/UserWithdrawalManager.sol
132:        for (requestId = nextRequestIdToFinalize; requestId <= maxRequestIdToFinalize; ) {
133:            UserWithdrawInfo memory userWithdrawInfo = userWithdrawRequests[requestId];
134:            uint256 requiredEth = userWithdrawInfo.ethExpected;
135:            uint256 lockedEthX = userWithdrawInfo.ethXAmount;
136:            uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);
137:            if (
138:                (ethToSendToFinalizeRequest + minEThRequiredToFinalizeRequest > pooledETH) ||
139:                (userWithdrawInfo.requestBlock + staderConfig.getMinBlockDelayToFinalizeWithdrawRequest() >
140:                    block.number)
141:            ) {
142:                break;
143:            }
144:            userWithdrawRequests[requestId].ethFinalized = minEThRequiredToFinalizeRequest;
145:            ethRequestedForWithdraw -= requiredEth; // @audit: potential sload + sstore on every iteration
```
```diff
diff --git a/contracts/UserWithdrawalManager.sol b/contracts/UserWithdrawalManager.sol
index f563434..d78095e 100644
--- a/contracts/UserWithdrawalManager.sol
+++ b/contracts/UserWithdrawalManager.sol
@@ -128,12 +128,12 @@ contract UserWithdrawalManager is
         uint256 lockedEthXToBurn;
         uint256 ethToSendToFinalizeRequest;
         uint256 requestId;
+        uint256 requiredEth;
         uint256 pooledETH = poolManager.balance;
         for (requestId = nextRequestIdToFinalize; requestId <= maxRequestIdToFinalize; ) {
             UserWithdrawInfo memory userWithdrawInfo = userWithdrawRequests[requestId];
-            uint256 requiredEth = userWithdrawInfo.ethExpected;
             uint256 lockedEthX = userWithdrawInfo.ethXAmount;
-            uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);
+            uint256 minEThRequiredToFinalizeRequest = Math.min(userWithdrawInfo.ethExpected, (lockedEthX * exchangeRate) / DECIMALS);
             if (
                 (ethToSendToFinalizeRequest + minEThRequiredToFinalizeRequest > pooledETH) ||
                 (userWithdrawInfo.requestBlock + staderConfig.getMinBlockDelayToFinalizeWithdrawRequest() >
@@ -142,7 +142,7 @@ contract UserWithdrawalManager is
                 break;
             }
             userWithdrawRequests[requestId].ethFinalized = minEThRequiredToFinalizeRequest;
-            ethRequestedForWithdraw -= requiredEth;
+            requiredEth += userWithdrawInfo.ethExpected;
             lockedEthXToBurn += lockedEthX;
             ethToSendToFinalizeRequest += minEThRequiredToFinalizeRequest;
             unchecked {
@@ -151,6 +151,7 @@ contract UserWithdrawalManager is
         }
         // at here, upto (requestId-1) is finalized
         if (requestId > nextRequestIdToFinalize) {
+            ethRequestedForWithdraw = ethRequestedForWithdraw - requiredEth;
             nextRequestIdToFinalize = requestId;
             ETHx(staderConfig.getETHxToken()).burnFrom(address(this), lockedEthXToBurn);
             IStaderStakePoolManager(poolManager).transferETHToUserWithdrawManager(ethToSendToFinalizeRequest);
```

## Cache external calls outside of loop to avoid re-calling function on each iteration
Performing `STATICCALL`s that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a `STATICCALL (100 gas)` per loop iteration.

Total Instances: `2`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L132-L139

### Cache `staderConfig.getMinBlockDelayToFinalizeWithdrawRequest()` outside of loop to save 1 `STATICCALL` per loop iteration

**Note: This will also save 1 SLOAD (`staderConfig`) per loop iteration. In addition, this function is not benchmarked in the protocol's tests and therefore the exact gas savings are not included here**.
```solidity
File: contracts/UserWithdrawalManager.sol
132:        for (requestId = nextRequestIdToFinalize; requestId <= maxRequestIdToFinalize; ) {
133:            UserWithdrawInfo memory userWithdrawInfo = userWithdrawRequests[requestId];
134:            uint256 requiredEth = userWithdrawInfo.ethExpected;
135:            uint256 lockedEthX = userWithdrawInfo.ethXAmount;
136:            uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);
137:            if (
138:                (ethToSendToFinalizeRequest + minEThRequiredToFinalizeRequest > pooledETH) ||
139:                (userWithdrawInfo.requestBlock + staderConfig.getMinBlockDelayToFinalizeWithdrawRequest() > // @audit: external call on every iteration
```
```diff
diff --git a/contracts/UserWithdrawalManager.sol b/contracts/UserWithdrawalManager.sol
index f563434..204239f 100644
--- a/contracts/UserWithdrawalManager.sol
+++ b/contracts/UserWithdrawalManager.sol
@@ -128,6 +128,7 @@ contract UserWithdrawalManager is
         uint256 lockedEthXToBurn;
         uint256 ethToSendToFinalizeRequest;
         uint256 requestId;
+        uint256 minBlockDelay = staderConfig.getMinBlockDelayToFinalizeWithdrawRequest();
         uint256 pooledETH = poolManager.balance;
         for (requestId = nextRequestIdToFinalize; requestId <= maxRequestIdToFinalize; ) {
             UserWithdrawInfo memory userWithdrawInfo = userWithdrawRequests[requestId];
@@ -136,7 +137,7 @@ contract UserWithdrawalManager is
             uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);
             if (
                 (ethToSendToFinalizeRequest + minEThRequiredToFinalizeRequest > pooledETH) ||
-                (userWithdrawInfo.requestBlock + staderConfig.getMinBlockDelayToFinalizeWithdrawRequest() >
+                (userWithdrawInfo.requestBlock + minBlockDelay >
                     block.number)
             ) {
                 break;
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L94-L109

### Cache return values from `INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID()` and `staderConfig.getETHDepositContract()` outisde of loop to save 3 `STATICCALL (100 GAS)` per iteration

**Note: Caching the `staderConfig.getPreDepositSize()` calls are not included since it was included in the automated report. In addition, this function is not benchmarked in the protocol's tests and therefore the exact gas savings are not included here**.

```solidity
File: contracts/PermissionlessPool.sol
94:        for (uint256 i; i < pubkeyCount; ) {
95:            address withdrawVault = IVaultFactory(vaultFactory).computeWithdrawVaultAddress(
96:                INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID(), // @audit: 2 external calls on every iteration
97:                _operatorId,
98:                _operatorTotalKeys + i
99:            );
100:            bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(withdrawVault);
101:
102:            bytes32 depositDataRoot = this.computeDepositDataRoot(
103:                _pubkey[i],
104:                _preDepositSignature[i],
105:                withdrawCredential,
106:                staderConfig.getPreDepositSize() 
107:            );
108:            //slither-disable-next-line arbitrary-send-eth
109:            IDepositContract(staderConfig.getETHDepositContract()).deposit{value: staderConfig.getPreDepositSize()}( // @audit: external call on every iteration
```
```diff
diff --git a/contracts/PermissionlessPool.sol b/contracts/PermissionlessPool.sol
index 046761d..ef0e3bf 100644
--- a/contracts/PermissionlessPool.sol
+++ b/contracts/PermissionlessPool.sol
@@ -83,17 +83,19 @@ contract PermissionlessPool is IStaderPoolBase, Initializable, AccessControlUpgr
      * @param _operatorTotalKeys total keys of operator at the starting of adding new keys
      */
     function preDepositOnBeaconChain(
-        bytes[] calldata _pubkey,
-        bytes[] calldata _preDepositSignature,
+        bytes[] memory _pubkey,
+        bytes[] memory _preDepositSignature,
         uint256 _operatorId,
         uint256 _operatorTotalKeys
     ) external payable nonReentrant {
         UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_NODE_REGISTRY());
         address vaultFactory = staderConfig.getVaultFactory();
         uint256 pubkeyCount = _pubkey.length;
+        uint8 poolId = INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID();
+        address ethDepositContract = staderConfig.getETHDepositContract();
         for (uint256 i; i < pubkeyCount; ) {
             address withdrawVault = IVaultFactory(vaultFactory).computeWithdrawVaultAddress(
-                INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID(),
+                poolId,
                 _operatorId,
                 _operatorTotalKeys + i
             );
@@ -106,7 +108,7 @@ contract PermissionlessPool is IStaderPoolBase, Initializable, AccessControlUpgr
                 staderConfig.getPreDepositSize()
             );
             //slither-disable-next-line arbitrary-send-eth
-            IDepositContract(staderConfig.getETHDepositContract()).deposit{value: staderConfig.getPreDepositSize()}(
+            IDepositContract(ethDepositContract).deposit{value: staderConfig.getPreDepositSize()}(
                 _pubkey[i],
                 withdrawCredential,
                 _preDepositSignature[i],
```

## Refactor code to avoid unnecessary SLOAD
Line 49 is essentially setting `balances[operator]` to 0. Instead of performing an extra SLOAD in the expression we can simply set the mapping to 0.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L46-L49

*Gas Savings for `OperatorRewardsCollector.claim`, obtained via protocol's tests: Avg 67 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  35757   |  35757   |  35757  |    3     |
| After  |  35690   |  35690   |  35690  |    3     |

```solidity
File: contracts/OperatorRewardsCollector.sol
46:    function claim() external whenNotPaused {
47:        address operator = msg.sender;
48:        uint256 amount = balances[operator]; // @audit: 1st sload
49:        balances[operator] -= amount; // @audit: 2nd sload
```
```diff
diff --git a/contracts/OperatorRewardsCollector.sol b/contracts/OperatorRewardsCollector.sol
index e80db8a..d284367 100644
--- a/contracts/OperatorRewardsCollector.sol
+++ b/contracts/OperatorRewardsCollector.sol
@@ -46,7 +46,7 @@ contract OperatorRewardsCollector is
     function claim() external whenNotPaused {
         address operator = msg.sender;
         uint256 amount = balances[operator];
-        balances[operator] -= amount;
+        balances[operator] = 0;

         address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig);
         UtilLib.sendValue(operatorRewardsAddr, amount);
```

## Utilize return value from internal function to avoid unnecessary SLOAD
The return value from the `onlyActiveOperator` internal function is `operatorIDByAddress[_operAddr]`. In the instances below, the return value is ignored and `operatorIDByAddress[msg.sender]` is being redundantly read from storage. Utilize the return value to save 1 SLOAD.

Total Instances: `2`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L401-L405

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L720-L721

### Cache return value from `onlyActiveOperator` inside of `operatorId` to save 1 SLOAD

```solidity
File: contracts/PermissionedNodeRegistry.sol
401:    function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {
402:        IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);
403:        UtilLib.checkNonZeroAddress(_rewardAddress);
404:        onlyActiveOperator(msg.sender); // @audit: 1st sload for `operatorIDByAddress[msg.sender]`
405:        uint256 operatorId = operatorIDByAddress[msg.sender]; // @audit: 2nd sload

720:    function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {
721:        _operatorId = operatorIDByAddress[_operAddr];
```
```diff
diff --git a/contracts/PermissionedNodeRegistry.sol b/contracts/PermissionedNodeRegistry.sol
index ca8aa81..80cfa7c 100644
--- a/contracts/PermissionedNodeRegistry.sol
+++ b/contracts/PermissionedNodeRegistry.sol
@@ -401,8 +401,7 @@ contract PermissionedNodeRegistry is
     function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {
         IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);
         UtilLib.checkNonZeroAddress(_rewardAddress);
-        onlyActiveOperator(msg.sender);
-        uint256 operatorId = operatorIDByAddress[msg.sender];
+        uint256 operatorId = onlyActiveOperator(msg.sender);
         operatorStructById[operatorId].operatorName = _operatorName;
         operatorStructById[operatorId].operatorRewardAddress = _rewardAddress;
         emit UpdatedOperatorDetails(msg.sender, _operatorName, _rewardAddress);
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L366-L370

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L680-L681

### Cache return value from `onlyActiveOperator` inside of `operatorId` to save 1 SLOAD

```solidity
File: contracts/PermissionlessNodeRegistry.sol
366:    function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {
367:        IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);
368:        UtilLib.checkNonZeroAddress(_rewardAddress);
369:        onlyActiveOperator(msg.sender); // @audit: 1st sload for `operatorIDByAddress[msg.sender]`
370:        uint256 operatorId = operatorIDByAddress[msg.sender]; // @audit: 2nd sload

680:    function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {
681:        _operatorId = operatorIDByAddress[_operAddr];
```
```diff
diff --git a/contracts/PermissionlessNodeRegistry.sol b/contracts/PermissionlessNodeRegistry.sol
index 9640556..5c06f22 100644
--- a/contracts/PermissionlessNodeRegistry.sol
+++ b/contracts/PermissionlessNodeRegistry.sol
@@ -366,8 +366,7 @@ contract PermissionlessNodeRegistry is
     function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {
         IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);
         UtilLib.checkNonZeroAddress(_rewardAddress);
-        onlyActiveOperator(msg.sender);
-        uint256 operatorId = operatorIDByAddress[msg.sender];
+        uint256 operatorId = onlyActiveOperator(msg.sender);
         operatorStructById[operatorId].operatorName = _operatorName;
         operatorStructById[operatorId].operatorRewardAddress = _rewardAddress;
         emit UpdatedOperatorDetails(msg.sender, _operatorName, _rewardAddress);
```

## Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate
We can combine multiple mappings below into structs. We can then pack the structs by modifying the `uint type` for the values. This will result in cheaper storage reads since multiple mappings are accessed in functions and those values are now occupying the same storage slot, meaning the slot will become warm after the first SLOAD. In addition, when writing to and reading from the struct values we will avoid a `Gsset (20000 gas)` and `Gcoldsload (2100 gas)` since multiple struct values are now occupying the same slot.

**Note: This instance was missed by the automated report**.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L30-L32

### Save at least two `Gsset (20000 gas)` when the `handleRewards` function is called (`handledRewards`, `poolId`, and `reportingBlockNumber` are all set in storage, but since they are contained in a single slot only one `Gsset (20000)` is paid.

***In order for this optimization to result in impactful gas savings we will need to pack the `poolId` and `reportingBlockNumber` in the `RewardsData` struct (we can safely reduce the `uint` type for `reportingBlockNumber` to `uint48`). We can then also pack the `handledRewards` variable with `poolId` and `reportingBlockNumber` by declaring it after the `RewardsData` struct in our newly created `RewardsInfo` struct.***
```solidity
File: contracts/SocializingPool.sol
30:    mapping(uint256 => bool) public handledRewards;
31:    RewardsData public lastReportedRewardsData;
32:    mapping(uint256 => RewardsData) public rewardsDataMap;
```
```diff
diff --git a/contracts/SocializingPool.sol b/contracts/SocializingPool.sol
index 19d8cb2..90b68de 100644
--- a/contracts/SocializingPool.sol
+++ b/contracts/SocializingPool.sol
@@ -26,10 +26,15 @@ contract SocializingPool is
     uint256 public override totalOperatorSDRewardsRemaining;
     uint256 public override initialBlock;

+    struct RewardsInfo {
+        RewardsData rewardsDataMap;
+        bool handledRewards;
+    }
+
+    mapping(uint256 => RewardsInfo) rewardsInfo;
+
     mapping(address => mapping(uint256 => bool)) public override claimedRewards;
-    mapping(uint256 => bool) public handledRewards;
     RewardsData public lastReportedRewardsData;
-    mapping(uint256 => RewardsData) public rewardsDataMap;

     /// @custom:oz-upgrades-unsafe-allow constructor
     constructor() {
@@ -60,8 +65,9 @@ contract SocializingPool is

     function handleRewards(RewardsData calldata _rewardsData) external override nonReentrant {
         UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.STADER_ORACLE());
-
-        if (handledRewards[_rewardsData.index]) {
+
+        RewardsInfo storage _rewardsInfo = rewardsInfo[_rewardsData.index];
+        if (_rewardsInfo.handledRewards) {
             revert RewardAlreadyHandled();
         }
         if (
@@ -77,12 +83,12 @@ contract SocializingPool is
             revert InsufficientSDRewards();
         }

-        handledRewards[_rewardsData.index] = true;
+        _rewardsInfo.handledRewards = true;
         totalOperatorETHRewardsRemaining += _rewardsData.operatorETHRewards;
         totalOperatorSDRewardsRemaining += _rewardsData.operatorSDRewards;

         lastReportedRewardsData = _rewardsData;
-        rewardsDataMap[_rewardsData.index] = _rewardsData;
+        _rewardsInfo.rewardsDataMap = _rewardsData;

         IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{
             value: _rewardsData.userETHRewards
@@ -170,7 +176,7 @@ contract SocializingPool is
         if (_index == 0 || _index > lastReportedRewardsData.index) {
             revert InvalidCycleIndex();
         }
-        bytes32 merkleRoot = rewardsDataMap[_index].merkleRoot;
+        bytes32 merkleRoot = rewardsInfo[_index].rewardsDataMap.merkleRoot;
         bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));
         return MerkleProofUpgradeable.verify(_merkleProof, merkleRoot, node);
     }
@@ -214,15 +220,16 @@ contract SocializingPool is
         }

         // for past cycles
-        _startBlock = rewardsDataMap[_index - 1].reportingBlockNumber + 1;
-        _endBlock = rewardsDataMap[_index].reportingBlockNumber;
+        _startBlock = rewardsInfo[_index - 1].rewardsDataMap.reportingBlockNumber + 1;
+        _endBlock = rewardsInfo[_index].rewardsDataMap.reportingBlockNumber;

         // for current cycle
-        if (rewardsDataMap[_index].reportingBlockNumber == 0) {
-            if (rewardsDataMap[_index - 1].reportingBlockNumber == 0) {
+        if (rewardsInfo[_index].rewardsDataMap.reportingBlockNumber == 0) {
+            if (rewardsInfo[_index - 1].rewardsDataMap.reportingBlockNumber == 0) {
                 revert FutureCycleIndex();
             }
-            _endBlock = rewardsDataMap[_index - 1].reportingBlockNumber + cycleDuration;
+            _endBlock = rewardsInfo[_index - 1].rewardsDataMap.reportingBlockNumber + cycleDuration;
         }
     }
 }
```
```diff
diff --git a/contracts/interfaces/ISocializingPool.sol b/contracts/interfaces/ISocializingPool.sol
index aceee8b..ab49093 100644
--- a/contracts/interfaces/ISocializingPool.sol
+++ b/contracts/interfaces/ISocializingPool.sol
@@ -7,14 +7,10 @@ import './IStaderConfig.sol';
 /// @title RewardsData
 /// @notice This struct holds rewards merkleRoot and rewards split
 struct RewardsData {
-    /// @notice The block number when the rewards data was last updated
-    uint256 reportingBlockNumber;
     /// @notice The index of merkle tree or rewards cycle
     uint256 index;
     /// @notice The merkle root hash
     bytes32 merkleRoot;
-    /// @notice pool id of operators
-    uint8 poolId;
     /// @notice operator ETH rewards for index cycle
     uint256 operatorETHRewards;
     /// @notice user ETH rewards for index cycle
@@ -23,6 +19,10 @@ struct RewardsData {
     uint256 protocolETHRewards;
     /// @notice operator SD rewards for index cycle
     uint256 operatorSDRewards;
+    /// @notice pool id of operators
+    uint8 poolId;
+    /// @notice The block number when the rewards data was last updated
+    uint48 reportingBlockNumber;
 }

 interface ISocializingPool {
```

## Using storage instead of memory for structs/arrays saves gas
The function in the instance below only reads 3 out of 5 struct fields. It is performing two unnecessary `Gcoldsload (2100 gas)`. We can change `memory` to `storage` in order to save those two SLOADs. Note that we will also need to cache the `owner` struct field since it is accessed multiple times.

**Note: This is an instance missed by the automated report**.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L169

```solidity
File: contracts/UserWithdrawalManager.sol
169:        UserWithdrawInfo memory userRequest = userWithdrawRequests[_requestId];
```
```diff
diff --git a/contracts/UserWithdrawalManager.sol b/contracts/UserWithdrawalManager.sol
index f563434..a12919f 100644
--- a/contracts/UserWithdrawalManager.sol
+++ b/contracts/UserWithdrawalManager.sol
@@ -166,8 +166,9 @@ contract UserWithdrawalManager is
         if (_requestId >= nextRequestIdToFinalize) {
             revert requestIdNotFinalized(_requestId);
         }
-        UserWithdrawInfo memory userRequest = userWithdrawRequests[_requestId];
-        if (msg.sender != userRequest.owner) {
+        UserWithdrawInfo storage userRequest = userWithdrawRequests[_requestId];
+        address payable _owner = userRequest.owner;
+        if (msg.sender != _owner) {
             revert CallerNotAuthorizedToRedeem();
         }
         // below is a default entry as no userRequest will be found for a redeemed request.
@@ -175,9 +176,9 @@ contract UserWithdrawalManager is
             revert RequestAlreadyRedeemed(_requestId);
         }
         uint256 etherToTransfer = userRequest.ethFinalized;
-        deleteRequestId(_requestId, userRequest.owner);
-        sendValue(userRequest.owner, etherToTransfer);
-        emit RequestRedeemed(msg.sender, userRequest.owner, etherToTransfer);
+        deleteRequestId(_requestId, _owner);
+        sendValue(_owner, etherToTransfer);
+        emit RequestRedeemed(msg.sender, _owner, etherToTransfer);
     }
```

## Using bools for storage incurs overhead
Both state variables can potentially be set back and forth from `true` and `false`. This would result in a `Gsset (20000 gas)` everytime the values are set to `true` from `false`. We can instead use `uint(1)` in place of `true` and `uint(2)` in place of `false` and pay the `Gsset (20000 gas)` once during deployment (to set the slot to `uint(1)`. This would save two `Gsset (20000 gas)`. However, a more efficient mitigation would be to pack the variables into a slot with other variables that would inevitably be written to. Please see [this finding](#pack-erinspectionmode-isporfeedbasederdata-erinspectionmodestartblock-staderconfig-and-exchangerate-into-one-slot-to-save-4-slots-8000-gas) for a more efficient solution.

**Note: This is an instance missed by the automated report**.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L18-L19

```solidity
File: contracts/StaderOracle.sol
18:    bool public override erInspectionMode;
19:    bool public override isPORFeedBasedERData;
```

## Multiple accesses of a mapping/array should use a local variable cache
Caching a mapping's value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `stakes[tokenId_]`, save its reference via a storage pointer: `StakeInfo storage stakeInfo = stakes[tokenId_]` and use the pointer instead.

Total Instances: `4`

**Note: These instances were missed by the automated report**.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L208-L209

### Use already cached `requestIdsByUserAddress[_owner]` instead of calling `requestIdsByUserAddress[_owner].length`

**Note: This function is not benchmarked in the protocol's tests and therefore exact gas savings are not included for this instance**.
```solidity
File: contracts/UserWithdrawalManager.sol
208:        uint256 userRequestCount = requestIdsByUserAddress[_owner].length;
209:        uint256[] storage requestIds = requestIdsByUserAddress[_owner];
```
```diff
diff --git a/contracts/UserWithdrawalManager.sol b/contracts/UserWithdrawalManager.sol
index f563434..664dce3 100644
--- a/contracts/UserWithdrawalManager.sol
+++ b/contracts/UserWithdrawalManager.sol
@@ -205,8 +205,8 @@ contract UserWithdrawalManager is
     // delete entry from userWithdrawRequests mapping and in requestIdsByUserAddress mapping
     function deleteRequestId(uint256 _requestId, address _owner) internal {
         delete (userWithdrawRequests[_requestId]);
-        uint256 userRequestCount = requestIdsByUserAddress[_owner].length;
         uint256[] storage requestIds = requestIdsByUserAddress[_owner];
+        uint256 userRequestCount = requestIds.length;
         for (uint256 i; i < userRequestCount; ) {
             if (_requestId == requestIds[i]) {
                 requestIds[i] = requestIds[userRequestCount - 1];
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L101-L107

### Cache `requestIdsByUserAddress[_owner]` as a storage pointer

**Note: This function is not benchmarked in the protocol's tests and therefore exact gas savings are not included for this instance**.
```solidity
File: contracts/UserWithdrawalManager.sol
101:        if (requestIdsByUserAddress[_owner].length + 1 > maxNonRedeemedUserRequestCount) {
102:            revert MaxLimitOnWithdrawRequestCountReached();
103:        }
104:        IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);
105:        ethRequestedForWithdraw += assets;
106:        userWithdrawRequests[nextRequestId] = UserWithdrawInfo(payable(_owner), _ethXAmount, assets, 0, block.number);
107:        requestIdsByUserAddress[_owner].push(nextRequestId);
```
```diff
diff --git a/contracts/UserWithdrawalManager.sol b/contracts/UserWithdrawalManager.sol
index f563434..c073ddb 100644
--- a/contracts/UserWithdrawalManager.sol
+++ b/contracts/UserWithdrawalManager.sol
@@ -98,13 +98,14 @@ contract UserWithdrawalManager is
         if (assets < staderConfig.getMinWithdrawAmount() || assets > staderConfig.getMaxWithdrawAmount()) {
             revert InvalidWithdrawAmount();
         }
-        if (requestIdsByUserAddress[_owner].length + 1 > maxNonRedeemedUserRequestCount) {
+        uint256[] storage _requestIdsByUserAddress = requestIdsByUserAddress[_owner];
+        if (_requestIdsByUserAddress.length + 1 > maxNonRedeemedUserRequestCount) {
             revert MaxLimitOnWithdrawRequestCountReached();
         }
         IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);
         ethRequestedForWithdraw += assets;
         userWithdrawRequests[nextRequestId] = UserWithdrawInfo(payable(_owner), _ethXAmount, assets, 0, block.number);
-        requestIdsByUserAddress[_owner].push(nextRequestId);
+        _requestIdsByUserAddress.push(nextRequestId);
         emit WithdrawRequestReceived(msg.sender, _owner, nextRequestId, _ethXAmount, assets);
         nextRequestId++;
         return nextRequestId - 1;
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L145-L155

### Cache `claimedRewards[_operator]` as a storage pointer outside of the for loop

**Note: In order to do this optimization we will need to first adopt the optimization in [This finding](#call-environment-variables-directly-instead-of-caching-them). We do this to avoid `stack too deep` errors. In addition, this function is not benchmarked in the protocol's tests and therefore exact gas savings are not included in this instance**.
```solidity
File: contracts/SocializingPool.sol
145:        for (uint256 i = 0; i < indexLength; i++) {
146:            if (_amountSD[i] == 0 && _amountETH[i] == 0) {
147:                revert InvalidAmount();
148:            }
149:            if (claimedRewards[_operator][_index[i]]) {
150:                revert RewardAlreadyClaimed(_operator, _index[i]);
151:            }
152:
153:            _totalAmountSD += _amountSD[i];
154:            _totalAmountETH += _amountETH[i];
155:            claimedRewards[_operator][_index[i]] = true;
```
```diff
diff --git a/contracts/SocializingPool.sol b/contracts/SocializingPool.sol
index 19d8cb2..ef5a343 100644
--- a/contracts/SocializingPool.sol
+++ b/contracts/SocializingPool.sol
@@ -111,7 +111,7 @@ contract SocializingPool is
         bytes32[][] calldata _merkleProof
     ) external override nonReentrant whenNotPaused {
         address operator = msg.sender;
-        (uint256 totalAmountSD, uint256 totalAmountETH) = _claim(_index, operator, _amountSD, _amountETH, _merkleProof);
+        (uint256 totalAmountSD, uint256 totalAmountETH) = _claim(_index, _amountSD, _amountETH, _merkleProof);

         address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(operator, staderConfig);

@@ -136,26 +136,26 @@ contract SocializingPool is

     function _claim(
         uint256[] calldata _index,
-        address _operator,
         uint256[] calldata _amountSD,
         uint256[] calldata _amountETH,
         bytes32[][] calldata _merkleProof
     ) internal returns (uint256 _totalAmountSD, uint256 _totalAmountETH) {
         uint256 indexLength = _index.length;
+        mapping(uint256 => bool) storage _claimedRewards = claimedRewards[msg.sender];
         for (uint256 i = 0; i < indexLength; i++) {
             if (_amountSD[i] == 0 && _amountETH[i] == 0) {
                 revert InvalidAmount();
             }
-            if (claimedRewards[_operator][_index[i]]) {
-                revert RewardAlreadyClaimed(_operator, _index[i]);
+            if (_claimedRewards[_index[i]]) {
+                revert RewardAlreadyClaimed(msg.sender, _index[i]);
             }

             _totalAmountSD += _amountSD[i];
             _totalAmountETH += _amountETH[i];
-            claimedRewards[_operator][_index[i]] = true;
+            _claimedRewards[_index[i]] = true;

-            if (!verifyProof(_index[i], _operator, _amountSD[i], _amountETH[i], _merkleProof[i])) {
-                revert InvalidProof(_index[i], _operator);
+            if (!verifyProof(_index[i], msg.sender, _amountSD[i], _amountETH[i], _merkleProof[i])) {
+                revert InvalidProof(_index[i], msg.sender);
             }
         }
     }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L78-L81

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L238-L242

*Gas Savings for `SDCollateral.getOperatorWithdrawThreshold`, obtained via protocol's tests: Avg 72 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  29012   |  29012   |  29012  |    2     |
| After  |  28940   |  28940   |  28940  |    2     |

**Note: This instance involves refactoring internal functions to pass in a storage pointer as a stack variable.**

### Pass cached `poolThreshold` storage pointers into internal function
```solidity
File: contracts/SDCollateral.sol
78:    function slashValidatorSD(uint256 _validatorId, uint8 _poolId) external override nonReentrant {
79:        address operator = UtilLib.getOperatorForValidSender(_poolId, _validatorId, msg.sender, staderConfig);
80:        isPoolThresholdValid(_poolId);
81:        PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];

238:    function isPoolThresholdValid(uint8 _poolId) internal view {
239:        if (bytes(poolThresholdbyPoolId[_poolId].units).length == 0) {
240:            revert InvalidPoolId();
241:        }
242:    }
```
```diff
diff --git a/contracts/SDCollateral.sol b/contracts/SDCollateral.sol
index 0b5e22b..0226693 100644
--- a/contracts/SDCollateral.sol
+++ bcontracts/SDCollateral.sol
@@ -77,8 +77,8 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
     /// @param _validatorId validator SD collateral to slash
     function slashValidatorSD(uint256 _validatorId, uint8 _poolId) external override nonReentrant {
         address operator = UtilLib.getOperatorForValidSender(_poolId, _validatorId, msg.sender, staderConfig);
-        isPoolThresholdValid(_poolId);
         PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];
+        isPoolThresholdValid(poolThreshold);
         uint256 sdToSlash = convertETHToSD(poolThreshold.minThreshold);
         slashSD(operator, sdToSlash);
     }
@@ -143,8 +143,8 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
     // returns sum of withdraw threshold accounting for all its(op's) validators
     function getOperatorWithdrawThreshold(address _operator) public view returns (uint256 operatorWithdrawThreshold) {
         (uint8 poolId, , uint256 validatorCount) = getOperatorInfo(_operator);
-        isPoolThresholdValid(poolId);
         PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[poolId];
+        isPoolThresholdValid(poolThreshold);
         return convertETHToSD(poolThreshold.withdrawThreshold * validatorCount);
     }

@@ -169,8 +169,8 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
         override
         returns (uint256 _minSDToBond)
     {
-        isPoolThresholdValid(_poolId);
         PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];
+        isPoolThresholdValid(poolThreshold);

         _minSDToBond = convertETHToSD(poolThreshold.minThreshold);
         _minSDToBond *= _numValidator;
@@ -193,8 +193,8 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
     function getRewardEligibleSD(address _operator) external view override returns (uint256 _rewardEligibleSD) {
         (uint8 poolId, , uint256 validatorCount) = getOperatorInfo(_operator);

-        isPoolThresholdValid(poolId);
         PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[poolId];
+        isPoolThresholdValid(poolThreshold);

         uint256 totalMinThreshold = validatorCount * convertETHToSD(poolThreshold.minThreshold);
         uint256 totalMaxThreshold = validatorCount * convertETHToSD(poolThreshold.maxThreshold);
@@ -235,9 +235,10 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
         );
     }

-    function isPoolThresholdValid(uint8 _poolId) internal view {
-        if (bytes(poolThresholdbyPoolId[_poolId].units).length == 0) {
+    function isPoolThresholdValid(PoolThresholdInfo storage poolThreshold) internal view {
+        if (bytes(poolThreshold.units).length == 0) {
             revert InvalidPoolId();
         }
     }
 }
```

## Combine events to save 2 `Glogtopic (375 gas)` 
The events below are only emitted once in the `handleRewards` function. We can combine the events into one singular event to save two `Glogtopic (375 gas)` that would otherwise be paid for the additional two events.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L96-L104

```solidity
File: contracts/SocializingPool.sol
96:        emit OperatorRewardsUpdated(
97:            _rewardsData.operatorETHRewards,
98:            totalOperatorETHRewardsRemaining,
99:            _rewardsData.operatorSDRewards,
100:            totalOperatorSDRewardsRemaining
101:        );
102:
103:        emit UserETHRewardsTransferred(_rewardsData.userETHRewards);
104:        emit ProtocolETHRewardsTransferred(_rewardsData.protocolETHRewards);
```

## Call environment variables directly instead of caching them
In the instance below, instead of caching `msg.sender` and incurring unnecessary stack manipulation, we can call `msg.sender` directly. `msg.sender` costs 2 Gas while the extra stack manipulation will cost 3 Gas per `DUP`. Note that the cached variable is used multiple times within a loop (see [`_claim`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L137-L161)).

**Note: This function is not benchmarked in the protocol's tests and therefore exact gas savings is not included for this instance**.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L107-L116

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L137-L161

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L163-L174

```solidity
File: contracts/SocializingPool.sol
106:    function claim(
107:        uint256[] calldata _index,
108:        uint256[] calldata _amountSD,
109:        uint256[] calldata _amountETH,
110:        bytes32[][] calldata _merkleProof
111:    ) external override nonReentrant whenNotPaused {
112:        address operator = msg.sender;
113:        (uint256 totalAmountSD, uint256 totalAmountETH) = _claim(_index, operator, _amountSD, _amountETH, _merkleProof);
114:
115:        address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(operator, staderConfig);

137:    function _claim(
138:        uint256[] calldata _index,
139:        address _operator, // @audit: cached msg.sender
140:        uint256[] calldata _amountSD,
141:        uint256[] calldata _amountETH,
142:        bytes32[][] calldata _merkleProof
143:    ) internal returns (uint256 _totalAmountSD, uint256 _totalAmountETH) {
144:        uint256 indexLength = _index.length;
145:        for (uint256 i = 0; i < indexLength; i++) {
146:            if (_amountSD[i] == 0 && _amountETH[i] == 0) {
147:                revert InvalidAmount();
148:            }
149:            if (claimedRewards[_operator][_index[i]]) {
150:                revert RewardAlreadyClaimed(_operator, _index[i]);
151:            }
152:
153:            _totalAmountSD += _amountSD[i];
154:            _totalAmountETH += _amountETH[i];
155:            claimedRewards[_operator][_index[i]] = true;
156:
157:            if (!verifyProof(_index[i], _operator, _amountSD[i], _amountETH[i], _merkleProof[i])) {
158:                revert InvalidProof(_index[i], _operator);
159:            }
160:        }
161:    }

163:    function verifyProof(
164:        uint256 _index,
165:        address _operator, // @audit: cached msg.sender
166:        uint256 _amountSD,
167:        uint256 _amountETH,
168:        bytes32[] calldata _merkleProof
169:    ) public view returns (bool) {
170:        if (_index == 0 || _index > lastReportedRewardsData.index) {
171:            revert InvalidCycleIndex();
172:        }
173:        bytes32 merkleRoot = rewardsDataMap[_index].merkleRoot;
174:        bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));
```
```diff
diff --git a/contracts/SocializingPool.sol b/contracts/SocializingPool.sol
index 19d8cb2..c504d2c 100644
--- a/contracts/SocializingPool.sol
+++ b/contracts/SocializingPool.sol
@@ -110,10 +110,9 @@ contract SocializingPool is
         uint256[] calldata _amountETH,
         bytes32[][] calldata _merkleProof
     ) external override nonReentrant whenNotPaused {
-        address operator = msg.sender;
-        (uint256 totalAmountSD, uint256 totalAmountETH) = _claim(_index, operator, _amountSD, _amountETH, _merkleProof);
+        (uint256 totalAmountSD, uint256 totalAmountETH) = _claim(_index, _amountSD, _amountETH, _merkleProof);

-        address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(operator, staderConfig);
+        address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig);

         bool success;
         if (totalAmountETH > 0) {
@@ -136,7 +135,6 @@ contract SocializingPool is

     function _claim(
         uint256[] calldata _index,
-        address _operator,
         uint256[] calldata _amountSD,
         uint256[] calldata _amountETH,
         bytes32[][] calldata _merkleProof
@@ -146,35 +144,44 @@ contract SocializingPool is
             if (_amountSD[i] == 0 && _amountETH[i] == 0) {
                 revert InvalidAmount();
             }
-            if (claimedRewards[_operator][_index[i]]) {
-                revert RewardAlreadyClaimed(_operator, _index[i]);
+            if (claimedRewards[msg.sender][_index[i]]) {
+                revert RewardAlreadyClaimed(msg.sender, _index[i]);
             }

             _totalAmountSD += _amountSD[i];
             _totalAmountETH += _amountETH[i];
-            claimedRewards[_operator][_index[i]] = true;
+            claimedRewards[msg.sender][_index[i]] = true;

-            if (!verifyProof(_index[i], _operator, _amountSD[i], _amountETH[i], _merkleProof[i])) {
-                revert InvalidProof(_index[i], _operator);
+            if (!_verifyProof(_index[i], _amountSD[i], _amountETH[i], _merkleProof[i])) {
+                revert InvalidProof(_index[i], msg.sender);
             }
         }
     }

-    function verifyProof(
+    function _verifyProof(
         uint256 _index,
-        address _operator,
         uint256 _amountSD,
         uint256 _amountETH,
         bytes32[] calldata _merkleProof
-    ) public view returns (bool) {
+    ) internal view returns (bool) {
         if (_index == 0 || _index > lastReportedRewardsData.index) {
             revert InvalidCycleIndex();
         }
         bytes32 merkleRoot = rewardsDataMap[_index].merkleRoot;
-        bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));
+        bytes32 node = keccak256(abi.encodePacked(msg.sender, _amountSD, _amountETH));
         return MerkleProofUpgradeable.verify(_merkleProof, merkleRoot, node);
     }

+    function verifyProof(
+        uint256 _index,
+        address _operator,
+        uint256 _amountSD,
+        uint256 _amountETH,
+        bytes32[] calldata _merkleProof
+    ) public view returns (bool) {
+        return _verifyProof(_index, _amountSD, _amountETH, _merkleProof);
+    }
+
     // SETTERS
     function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {
         UtilLib.checkNonZeroAddress(_staderConfig);
```

## Use assembly to perform efficient back-to-back calls
If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (`scratch space` + `free memory pointer`), which can potentially allow us to avoid memory expansion costs. In this case we are also able to efficiently store the function signatures together in memory as one word, saving multiple `MLOAD`s in the process.

Total Instances: `3`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L55-L58

*Gas Savings for `ValidatorWithdrawalVault.settleFunds`, obtained via protocol's tests: Avg 1320 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  182196  |  196636  |  130849 |    3     |
| After  |  180876  |  195316  |  129529 |    3     |

```solidity
File: contracts/ValidatorWithdrawalVault.sol
55:        uint8 poolId = VaultProxy(payable(address(this))).poolId();
56:        uint256 validatorId = VaultProxy(payable(address(this))).id();
57:        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
58:        address nodeRegistry = IPoolUtils(staderConfig.getPoolUtils()).getNodeRegistry(poolId);
```
```diff
diff --git a/contracts/ValidatorWithdrawalVault.sol b/contracts/ValidatorWithdrawalVault.sol
index eb3de50..a2e6714 100644
--- a/contracts/ValidatorWithdrawalVault.sol
+++ b/contracts/ValidatorWithdrawalVault.sol
@@ -52,10 +52,25 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
     }

     function settleFunds() external override {
-        uint8 poolId = VaultProxy(payable(address(this))).poolId();
-        uint256 validatorId = VaultProxy(payable(address(this))).id();
-        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
-        address nodeRegistry = IPoolUtils(staderConfig.getPoolUtils()).getNodeRegistry(poolId);
+        uint8 poolId;
+        uint256 validatorId;
+        IStaderConfig staderConfig;
+        address nodeRegistry;
+        assembly {
+            // sigs for "getNodeRegistry(uint8)" + "poolId()" + "id()" + "staderConfig()" + "getPoolUtils()"
+            mstore(0x00, 0x6ccb9d70490ffa35af640d0f3e0dc34e99d055c8)
+            if iszero(staticcall(gas(), address(), 0x18, 0x04, 0x20, 0x20)) {revert(0, 0)}
+            poolId := mload(0x20)
+            if iszero(staticcall(gas(), address(), 0x14, 0x04, 0x20, 0x20)) {revert(0, 0)}
+            validatorId := mload(0x20)
+            if iszero(staticcall(gas(), address(), 0x10, 0x04, 0x20, 0x20)) {revert(0, 0)}
+            staderConfig := mload(0x20)
+            if iszero(staticcall(gas(), staderConfig, 0xc, 0x04, 0x20, 0x20)) {revert(0, 0)}
+            let poolUtils := mload(0x20)
+            mstore(0x20, poolId)
+            if iszero(staticcall(gas(), poolUtils, 0x1c, 0x24, 0x20, 0x20)) {revert(0, 0)}
+            nodeRegistry := mload(0x20)
+        }
         if (msg.sender != nodeRegistry) {
             revert CallerNotNodeRegistryContract();
         }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L31-L33C73

*Gas Savings for `ValidatorWithdrawalVault.distributeRewards`, obtained via protocol's tests: Avg 736 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  83852   |  161928  |  85468  |    4     |
| After  |  83115   |  161192  |  84732  |    4     |

```solidity
File: contracts/ValidatorWithdrawalVault.sol
31:        uint8 poolId = VaultProxy(payable(address(this))).poolId();
32:        uint256 validatorId = VaultProxy(payable(address(this))).id();
33:        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
```
```diff
diff --git a/contracts/ValidatorWithdrawalVault.sol b/contracts/ValidatorWithdrawalVault.sol
index eb3de50..8abfbc8 100644
--- a/contracts/ValidatorWithdrawalVault.sol
+++ b/contracts/ValidatorWithdrawalVault.sol
@@ -28,9 +28,19 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
     }

     function distributeRewards() external override {
-        uint8 poolId = VaultProxy(payable(address(this))).poolId();
-        uint256 validatorId = VaultProxy(payable(address(this))).id();
-        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
+        uint8 poolId;
+        uint256 validatorId;
+        IStaderConfig staderConfig;
+        assembly {
+            // sigs for "poolId()" + "id()" + "staderConfig()"
+            mstore(0x00, 0x490ffa35af640d0f3e0dc34e)
+            if iszero(staticcall(gas(), address(), 0x1c, 0x04, 0x20, 0x20)) {revert(0, 0)}
+            poolId := mload(0x20)
+            if iszero(staticcall(gas(), address(), 0x18, 0x04, 0x20, 0x20)) {revert(0, 0)}
+            validatorId := mload(0x20)
+            if iszero(staticcall(gas(), address(), 0x14, 0x04, 0x20, 0x20)) {revert(0, 0)}
+            staderConfig := mload(0x20)
+        }
         uint256 totalRewards = address(this).balance;
         if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
             emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L25-L27

*Gas Savings for `NodeELRewardVault.withdraw`, obtained via protocol's tests: Max 733 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  81428   |  122581  |  69510  |    3     |
| After  |  80695   |  121848  |  68777  |    3     |

```solidity
File: contracts/NodeELRewardVault.sol
25:        uint8 poolId = IVaultProxy(address(this)).poolId();
26:        uint256 operatorId = IVaultProxy(address(this)).id();
27:        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
```
```diff
diff --git a/contracts/NodeELRewardVault.sol b/contracts/NodeELRewardVault.sol 
index b10417a..77b2db3 100644
--- a/contracts/NodeELRewardVault.sol 
+++ b/contracts/NodeELRewardVault.sol 
@@ -22,9 +22,19 @@ contract NodeELRewardVault is INodeELRewardVault {
     }

     function withdraw() external override {
-        uint8 poolId = IVaultProxy(address(this)).poolId();
-        uint256 operatorId = IVaultProxy(address(this)).id();
-        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
+        uint8 poolId;
+        uint256 operatorId;
+        IStaderConfig staderConfig;
+        assembly {
+            // function sigs for "poolId()", "id()", and "staderConfig()"
+            mstore(0x00, 0x490ffa35af640d0f3e0dc34e)
+            if iszero(staticcall(gas(), address(), 0x1c, 0x04, 0x20, 0x20)) {revert(0, 0)}
+            poolId := mload(0x20)
+            if iszero(staticcall(gas(), address(), 0x18, 0x04, 0x20, 0x20)) {revert(0, 0)}
+            operatorId := mload(0x20)
+            if iszero(staticcall(gas(), address(), 0x14, 0x04, 0x20, 0x20)) {revert(0, 0)}
+            staderConfig := mload(0x20)
+        }
         uint256 totalRewards = address(this).balance;
         if (totalRewards == 0) {
             revert NotEnoughRewardToWithdraw();
```

## `GasReport` output with all optimizations applied
```js
| contracts/Auction.sol:Auction contract |                 |        |        |        |         |
|----------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                        | Deployment Size |        |        |        |         |
| 1535267                                | 7788            |        |        |        |         |
| Function Name                          | min             | avg    | median | max    | # calls |
| DEFAULT_ADMIN_ROLE                     | 328             | 328    | 328    | 328    | 1       |
| MIN_AUCTION_DURATION                   | 261             | 261    | 261    | 261    | 3       |
| addBid                                 | 478             | 42495  | 69177  | 69177  | 15      |
| bidIncrement                           | 339             | 456    | 339    | 2339   | 17      |
| claimSD                                | 547             | 8641   | 696    | 40589  | 5       |
| createLot                              | 61788           | 103958 | 110058 | 112058 | 16      |
| duration                               | 341             | 507    | 341    | 2341   | 12      |
| extractNonBidSD                        | 482             | 6265   | 608    | 29020  | 5       |
| hasRole                                | 2742            | 2742   | 2742   | 2742   | 1       |
| initialize                             | 144459          | 144459 | 144459 | 144459 | 39      |
| lots                                   | 1342            | 3495   | 3342   | 7342   | 13      |
| nextLot                                | 465             | 1215   | 465    | 2465   | 8       |
| staderConfig                           | 426             | 1426   | 1426   | 2426   | 2       |
| transferHighestBidToSSPM               | 3809            | 12515  | 7643   | 37839  | 5       |
| updateBidIncrement                     | 21621           | 21621  | 21621  | 21621  | 1       |
| updateDuration                         | 15564           | 18578  | 18578  | 21592  | 2       |
| updateStaderConfig                     | 9064            | 9064   | 9064   | 9064   | 1       |
| withdrawUnselectedBid                  | 3928            | 10565  | 5579   | 32016  | 5       |


| contracts/ETHx.sol:ETHx contract |                 |        |        |        |         |
|----------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                  | Deployment Size |        |        |        |         |
| 1332855                          | 6777            |        |        |        |         |
| Function Name                    | min             | avg    | median | max    | # calls |
| BURNER_ROLE                      | 307             | 307    | 307    | 307    | 2       |
| DEFAULT_ADMIN_ROLE               | 262             | 262    | 262    | 262    | 1       |
| MINTER_ROLE                      | 261             | 261    | 261    | 261    | 3       |
| balanceOf                        | 627             | 1427   | 627    | 2627   | 5       |
| burnFrom                         | 1005            | 8825   | 3622   | 32255  | 5       |
| grantRole                        | 27527           | 29678  | 29527  | 34433  | 6       |
| hasRole                          | 2743            | 2743   | 2743   | 2743   | 1       |
| initialize                       | 120650          | 120650 | 120650 | 120650 | 7       |
| mint                             | 1049            | 30205  | 39860  | 49421  | 6       |
| pause                            | 15425           | 22545  | 15425  | 36787  | 3       |
| staderConfig                     | 383             | 1716   | 2383   | 2383   | 3       |
| totalSupply                      | 2327            | 2327   | 2327   | 2327   | 1       |
| unpause                          | 1651            | 16874  | 16874  | 32098  | 2       |
| updateStaderConfig               | 7004            | 19619  | 19619  | 32234  | 2       |


| contracts/NodeELRewardVault.sol:NodeELRewardVault contract |                 |       |        |        |         |
|------------------------------------------------------------|-----------------|-------|--------|--------|---------|
| Deployment Cost                                            | Deployment Size |       |        |        |         |
| 380818                                                     | 1934            |       |        |        |         |
| Function Name                                              | min             | avg   | median | max    | # calls |
| receive                                                    | 1491            | 1491  | 1491   | 1491   | 1       |
| withdraw                                                   | 3788            | 67654 | 79009  | 120166 | 3       |


| contracts/OperatorRewardsCollector.sol:OperatorRewardsCollector contract |                 |       |        |       |         |
|--------------------------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                                          | Deployment Size |       |        |       |         |
| 889795                                                                   | 4564            |       |        |       |         |
| Function Name                                                            | min             | avg   | median | max   | # calls |
| balances                                                                 | 581             | 1025  | 581    | 2581  | 9       |
| claim                                                                    | 35690           | 35690 | 35690  | 35690 | 3       |
| depositFor                                                               | 4564            | 19684 | 22464  | 24464 | 5       |
| initialize                                                               | 75165           | 75165 | 75165  | 75165 | 18      |


| contracts/Penalty.sol:Penalty contract  |                 |        |        |        |         |
|-----------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                         | Deployment Size |        |        |        |         |
| 1632579                                 | 8274            |        |        |        |         |
| Function Name                           | min             | avg    | median | max    | # calls |
| DEFAULT_ADMIN_ROLE                      | 262             | 262    | 262    | 262    | 1       |
| calculateMEVTheftPenalty                | 1693            | 3611   | 3611   | 5529   | 2       |
| calculateMissedAttestationPenalty       | 17840           | 17840  | 17840  | 17840  | 1       |
| hasRole                                 | 2676            | 2676   | 2676   | 2676   | 1       |
| initialize                              | 183595          | 183595 | 183595 | 183595 | 12      |
| markValidatorSettled                    | 9552            | 9669   | 9669   | 9787   | 2       |
| mevTheftPenaltyPerStrike                | 384             | 955    | 384    | 2384   | 7       |
| missedAttestationPenaltyPerStrike       | 340             | 1006   | 340    | 2340   | 6       |
| ratedOracleAddress                      | 383             | 1716   | 2383   | 2383   | 3       |
| setAdditionalPenaltyAmount              | 15744           | 15775  | 15775  | 15807  | 2       |
| staderConfig                            | 382             | 1382   | 1382   | 2382   | 2       |
| totalPenaltyAmount                      | 971             | 971    | 971    | 971    | 3       |
| updateMEVTheftPenaltyPerStrike          | 15502           | 17478  | 17478  | 19454  | 2       |
| updateMissedAttestationPenaltyPerStrike | 15568           | 17544  | 17544  | 19520  | 2       |
| updateRatedOracleAddress                | 15578           | 16986  | 15644  | 19737  | 3       |
| updateStaderConfig                      | 8912            | 20592  | 20592  | 32273  | 2       |
| updateTotalPenaltyAmount                | 21970           | 51089  | 47151  | 84147  | 3       |
| updateValidatorExitPenaltyThreshold     | 15546           | 17522  | 17522  | 19498  | 2       |
| validatorExitPenaltyThreshold           | 384             | 1717   | 2384   | 2384   | 3       |


| contracts/PermissionedNodeRegistry.sol:PermissionedNodeRegistry contract |                 |        |         |         |         |
|--------------------------------------------------------------------------|-----------------|--------|---------|---------|---------|
| Deployment Cost                                                          | Deployment Size |        |         |         |         |
| 4125391                                                                  | 20730           |        |         |         |         |
| Function Name                                                            | min             | avg    | median  | max     | # calls |
| DEFAULT_ADMIN_ROLE                                                       | 262             | 262    | 262     | 262     | 1       |
| activateNodeOperator                                                     | 2725            | 13491  | 13491   | 24257   | 2       |
| addValidatorKeys                                                         | 1674            | 962403 | 1444691 | 1444691 | 15      |
| allocateValidatorsAndUpdateOperatorId                                    | 7984            | 10053  | 10095   | 12039   | 4       |
| deactivateNodeOperator                                                   | 2743            | 3243   | 3493    | 3493    | 3       |
| getAllActiveValidators                                                   | 401             | 1128   | 1128    | 1855    | 2       |
| getCollateralETH                                                         | 325             | 325    | 325     | 325     | 1       |
| getOperatorRewardAddress                                                 | 574             | 574    | 574     | 574     | 1       |
| getOperatorTotalKeys                                                     | 528             | 528    | 528     | 528     | 2       |
| getOperatorTotalNonTerminalKeys                                          | 519             | 772    | 772     | 1026    | 2       |
| getSocializingPoolStateChangeBlock                                       | 504             | 504    | 504     | 504     | 1       |
| getTotalActiveValidatorCount                                             | 403             | 403    | 403     | 403     | 1       |
| getTotalQueuedValidatorCount                                             | 3278            | 3993   | 3278    | 6140    | 4       |
| getValidatorsByOperator                                                  | 538             | 8101   | 3015    | 20750   | 3       |
| hasRole                                                                  | 2787            | 2787   | 2787    | 2787    | 1       |
| increaseTotalActiveValidatorCount                                        | 16960           | 27416  | 27308   | 38308   | 6       |
| initialize                                                               | 207030          | 207030 | 207030  | 207030  | 40      |
| inputKeyCountLimit                                                       | 402             | 402    | 402     | 402     | 2       |
| isExistingOperator                                                       | 664             | 1664   | 1664    | 2664    | 2       |
| isExistingPubkey                                                         | 907             | 907    | 907     | 907     | 1       |
| markValidatorReadyToDeposit                                              | 7654            | 11832  | 8859    | 18983   | 3       |
| markValidatorStatusAsPreDeposit                                          | 26713           | 26713  | 26713   | 26713   | 6       |
| maxNonTerminalKeyPerOperator                                             | 416             | 416    | 416     | 416     | 2       |
| nextOperatorId                                                           | 2430            | 2430   | 2430    | 2430    | 1       |
| nextQueuedValidatorIndexByOperatorId                                     | 571             | 571    | 571     | 571     | 2       |
| nextValidatorId                                                          | 405             | 1071   | 405     | 2405    | 3       |
| onboardNodeOperator                                                      | 930             | 162710 | 190474  | 233865  | 26      |
| onlyPreDepositValidator                                                  | 1102            | 1111   | 1111    | 1120    | 2       |
| operatorIDByAddress                                                      | 580             | 580    | 580     | 580     | 9       |
| operatorIdForExcessDeposit                                               | 407             | 807    | 407     | 2407    | 5       |
| operatorStructById                                                       | 2315            | 2315   | 2315    | 2315    | 1       |
| pause                                                                    | 38805           | 38805  | 38805   | 38805   | 1       |
| socializingPoolStateChangeBlock                                          | 551             | 551    | 551     | 551     | 1       |
| staderConfig                                                             | 467             | 1467   | 1467    | 2467    | 2       |
| totalActiveValidatorCount                                                | 385             | 1051   | 385     | 2385    | 3       |
| unpause                                                                  | 3107            | 3107   | 3107    | 3107    | 1       |
| updateDepositStatusAndBlock                                              | 25666           | 25666  | 25666   | 25666   | 7       |
| updateInputKeyCountLimit                                                 | 15590           | 17631  | 17631   | 19672   | 2       |
| updateMaxNonTerminalKeyPerOperator                                       | 15610           | 18358  | 19732   | 19732   | 3       |
| updateMaxOperatorId                                                      | 3002            | 5762   | 5762    | 8522    | 2       |
| updateOperatorDetails                                                    | 3685            | 9351   | 6099    | 21523   | 4       |
| updateQueuedValidatorIndex                                               | 25472           | 25997  | 25472   | 27572   | 4       |
| updateStaderConfig                                                       | 9009            | 20697  | 20697   | 32386   | 2       |
| updateVerifiedKeysBatchSize                                              | 2983            | 15453  | 18602   | 21628   | 4       |
| validatorIdByPubkey                                                      | 889             | 889    | 889     | 889     | 12      |
| validatorRegistry                                                        | 5517            | 5517   | 5517    | 5517    | 3       |
| verifiedKeyBatchSize                                                     | 384             | 1384   | 1384    | 2384    | 2       |
| whitelistPermissionedNOs                                                 | 26475           | 29278  | 26475   | 39475   | 51      |
| withdrawnValidators                                                      | 6790            | 14323  | 8010    | 28170   | 3       |


| contracts/PermissionlessNodeRegistry.sol:PermissionlessNodeRegistry contract |                 |        |         |         |         |
|------------------------------------------------------------------------------|-----------------|--------|---------|---------|---------|
| Deployment Cost                                                              | Deployment Size |        |         |         |         |
| 4399552                                                                      | 22099           |        |         |         |         |
| Function Name                                                                | min             | avg    | median  | max     | # calls |
| DEFAULT_ADMIN_ROLE                                                           | 262             | 262    | 262     | 262     | 1       |
| POOL_ID                                                                      | 337             | 337    | 337     | 337     | 1       |
| addValidatorKeys                                                             | 4725            | 829233 | 1444464 | 1444464 | 14      |
| changeSocializingPoolState                                                   | 1109            | 26963  | 5300    | 96145   | 4       |
| getAllActiveValidators                                                       | 401             | 1128   | 1128    | 1855    | 2       |
| getCollateralETH                                                             | 325             | 325    | 325     | 325     | 1       |
| getNodeELVaultAddressForOptOutOperators                                      | 423             | 80465  | 80465   | 160508  | 2       |
| getOperatorRewardAddress                                                     | 552             | 552    | 552     | 552     | 1       |
| getOperatorTotalKeys                                                         | 505             | 505    | 505     | 505     | 2       |
| getOperatorTotalNonTerminalKeys                                              | 497             | 750    | 750     | 1004    | 2       |
| getSocializingPoolStateChangeBlock                                           | 504             | 504    | 504     | 504     | 2       |
| getTotalActiveValidatorCount                                                 | 403             | 403    | 403     | 403     | 1       |
| getTotalQueuedValidatorCount                                                 | 2626            | 2626   | 2626    | 2626    | 3       |
| getValidatorsByOperator                                                      | 471             | 8034   | 2948    | 20683   | 3       |
| hasRole                                                                      | 2787            | 2787   | 2787    | 2787    | 1       |
| increaseTotalActiveValidatorCount                                            | 16922           | 27400  | 27270   | 38270   | 5       |
| initialize                                                                   | 162821          | 162821 | 162821  | 162821  | 47      |
| inputKeyCountLimit                                                           | 424             | 424    | 424     | 424     | 2       |
| isExistingOperator                                                           | 664             | 664    | 664     | 664     | 1       |
| isExistingPubkey                                                             | 930             | 930    | 930     | 930     | 1       |
| markValidatorReadyToDeposit                                                  | 6888            | 81623  | 76257   | 167903  | 5       |
| maxNonTerminalKeyPerOperator                                                 | 461             | 461    | 461     | 461     | 2       |
| nextOperatorId                                                               | 364             | 1364   | 1364    | 2364    | 2       |
| nextQueuedValidatorIndex                                                     | 384             | 384    | 384     | 384     | 1       |
| nextValidatorId                                                              | 383             | 1049   | 383     | 2383    | 3       |
| nodeELRewardVaultByOperatorId                                                | 545             | 545    | 545     | 545     | 4       |
| onboardNodeOperator                                                          | 988             | 307945 | 339115  | 379631  | 34      |
| operatorIDByAddress                                                          | 624             | 624    | 624     | 624     | 6       |
| operatorStructById                                                           | 2292            | 2292   | 2292    | 2292    | 2       |
| pause                                                                        | 38879           | 38879  | 38879   | 38879   | 1       |
| socializingPoolStateChangeBlock                                              | 529             | 529    | 529     | 529     | 2       |
| staderConfig                                                                 | 488             | 1488   | 1488    | 2488    | 2       |
| totalActiveValidatorCount                                                    | 363             | 1363   | 1363    | 2363    | 2       |
| transferCollateralToPool                                                     | 22019           | 25710  | 25710   | 29401   | 2       |
| unpause                                                                      | 3147            | 3147   | 3147    | 3147    | 1       |
| updateDepositStatusAndBlock                                                  | 5750            | 14278  | 5750    | 25650   | 7       |
| updateInputKeyCountLimit                                                     | 15641           | 17682  | 17682   | 19723   | 2       |
| updateMaxNonTerminalKeyPerOperator                                           | 15639           | 18387  | 19761   | 19761   | 3       |
| updateNextQueuedValidatorIndex                                               | 40054           | 40054  | 40054   | 40054   | 1       |
| updateOperatorDetails                                                        | 3663            | 10953  | 4871    | 30407   | 4       |
| updateStaderConfig                                                           | 8987            | 20675  | 20675   | 32364   | 2       |
| updateVerifiedKeysBatchSize                                                  | 3006            | 15481  | 18631   | 21657   | 4       |
| validatorIdByPubkey                                                          | 889             | 889    | 889     | 889     | 12      |
| validatorRegistry                                                            | 5495            | 5495   | 5495    | 5495    | 3       |
| verifiedKeyBatchSize                                                         | 384             | 1384   | 1384    | 2384    | 2       |
| withdrawnValidators                                                          | 8061            | 18608  | 10574   | 37189   | 3       |


| contracts/SDCollateral.sol:SDCollateral contract |                 |       |        |        |         |
|--------------------------------------------------|-----------------|-------|--------|--------|---------|
| Deployment Cost                                  | Deployment Size |       |        |        |         |
| 1972154                                          | 9977            |       |        |        |         |
| Function Name                                    | min             | avg   | median | max    | # calls |
| DEFAULT_ADMIN_ROLE                               | 306             | 306   | 306    | 306    | 1       |
| convertETHToSD                                   | 4422            | 12047 | 10922  | 21922  | 4       |
| convertSDToETH                                   | 4502            | 4502  | 4502   | 4502   | 1       |
| depositSDAsCollateral                            | 38774           | 49359 | 50937  | 55897  | 7       |
| getMinimumSDToBond                               | 11544           | 11544 | 11544  | 11544  | 1       |
| getOperatorWithdrawThreshold                     | 28675           | 28675 | 28675  | 28675  | 2       |
| getRemainingSDToBond                             | 5122            | 5358  | 5358   | 5594   | 2       |
| getRewardEligibleSD                              | 26798           | 26798 | 26798  | 26798  | 1       |
| hasEnoughSDCollateral                            | 5663            | 5663  | 5663   | 5663   | 1       |
| hasRole                                          | 2698            | 2698  | 2698   | 2698   | 1       |
| initialize                                       | 95075           | 95075 | 95075  | 95075  | 20      |
| maxApproveSD                                     | 31330           | 31330 | 31330  | 31330  | 1       |
| operatorSDBalance                                | 623             | 845   | 623    | 2623   | 9       |
| poolThresholdbyPoolId                            | 1885            | 1885  | 1885   | 1885   | 1       |
| slashValidatorSD                                 | 50750           | 93733 | 73846  | 169433 | 6       |
| staderConfig                                     | 448             | 1448  | 1448   | 2448   | 2       |
| updatePoolThreshold                              | 16059           | 77570 | 106758 | 106758 | 10      |
| updateStaderConfig                               | 5026            | 15512 | 9195   | 32317  | 3       |
| withdraw                                         | 11363           | 21169 | 19050  | 33094  | 3       |


| contracts/StaderConfig.sol:StaderConfig contract |                 |        |        |        |         |
|--------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                  | Deployment Size |        |        |        |         |
| 2758061                                          | 13902           |        |        |        |         |
| Function Name                                    | min             | avg    | median | max    | # calls |
| MANAGER                                          | 364             | 364    | 364    | 364    | 162     |
| OPERATOR                                         | 285             | 285    | 285    | 285    | 86      |
| PERMISSIONED_POOL                                | 330             | 330    | 330    | 330    | 29      |
| PERMISSIONLESS_POOL                              | 285             | 285    | 285    | 285    | 15      |
| STADER_ORACLE                                    | 308             | 308    | 308    | 308    | 6       |
| getAdmin                                         | 495             | 1181   | 495    | 2495   | 102     |
| getAuctionContract                               | 506             | 1006   | 506    | 2506   | 8       |
| getDecimals                                      | 399             | 1176   | 399    | 2399   | 18      |
| getETHxToken                                     | 2460            | 2460   | 2460   | 2460   | 1       |
| getNodeELRewardVaultImplementation               | 2482            | 2482   | 2482   | 2482   | 4       |
| getOperatorRewardsCollector                      | 2484            | 2484   | 2484   | 2484   | 7       |
| getPenaltyContract                               | 439             | 1772   | 2439   | 2439   | 3       |
| getPermissionedPool                              | 505             | 1484   | 505    | 2505   | 49      |
| getPermissionedSocializingPool                   | 460             | 2364   | 2460   | 2460   | 21      |
| getPermissionlessPool                            | 483             | 1222   | 483    | 2483   | 73      |
| getPermissionlessSocializingPool                 | 2474            | 2474   | 2474   | 2474   | 16      |
| getPoolSelector                                  | 2461            | 2461   | 2461   | 2461   | 1       |
| getPoolUtils                                     | 485             | 1658   | 2485   | 2485   | 121     |
| getPreDepositSize                                | 469             | 2246   | 2469   | 2469   | 9       |
| getRewardsThreshold                              | 467             | 967    | 467    | 2467   | 4       |
| getSDCollateral                                  | 440             | 2344   | 2440   | 2440   | 21      |
| getSocializingPoolOptInCoolingPeriod             | 401             | 1067   | 401    | 2401   | 3       |
| getStaderInsuranceFund                           | 2461            | 2461   | 2461   | 2461   | 1       |
| getStaderOracle                                  | 461             | 1318   | 461    | 2461   | 21      |
| getStaderToken                                   | 483             | 1983   | 2483   | 2483   | 28      |
| getStaderTreasury                                | 440             | 690    | 440    | 2440   | 8       |
| getStakePoolManager                              | 505             | 755    | 505    | 2505   | 8       |
| getStakedEthPerNode                              | 464             | 1214   | 464    | 2464   | 16      |
| getTotalFee                                      | 422             | 977    | 422    | 2422   | 18      |
| getValidatorWithdrawalVaultImplementation        | 507             | 1801   | 2507   | 2507   | 17      |
| getVaultFactory                                  | 441             | 1828   | 2441   | 2441   | 49      |
| getWithdrawnKeyBatchSize                         | 500             | 500    | 500    | 500    | 6       |
| grantRole                                        | 27573           | 27581  | 27573  | 29573  | 248     |
| initialize                                       | 355923          | 355923 | 355923 | 355923 | 162     |
| onlyManagerRole                                  | 815             | 1864   | 2815   | 2815   | 101     |
| onlyOperatorRole                                 | 828             | 2058   | 2828   | 2828   | 26      |
| onlyStaderContract                               | 703             | 1183   | 703    | 2703   | 50      |
| updateAdmin                                      | 27444           | 27444  | 27444  | 27444  | 16      |
| updateAuctionContract                            | 24445           | 24445  | 24445  | 24445  | 19      |
| updateETHxToken                                  | 24521           | 24521  | 24521  | 24521  | 6       |
| updateNodeELRewardImplementation                 | 24488           | 24488  | 24488  | 24488  | 91      |
| updateOperatorRewardsCollector                   | 24489           | 24489  | 24489  | 24489  | 101     |
| updatePenaltyContract                            | 24464           | 24464  | 24464  | 24464  | 106     |
| updatePermissionedPool                           | 24444           | 24494  | 24444  | 26444  | 40      |
| updatePermissionedSocializingPool                | 24443           | 24443  | 24443  | 24443  | 39      |
| updatePermissionlessPool                         | 24488           | 24488  | 24488  | 24488  | 46      |
| updatePermissionlessSocializingPool              | 24468           | 24468  | 24468  | 24468  | 46      |
| updatePoolUtils                                  | 24467           | 24467  | 24467  | 24467  | 131     |
| updateRewardsThreshold                           | 26361           | 26361  | 26361  | 26361  | 2       |
| updateSDCollateral                               | 24490           | 24490  | 24490  | 24490  | 95      |
| updateSocializingPoolOptInCoolingPeriod          | 26317           | 26317  | 26317  | 26317  | 2       |
| updateStaderInsuranceFund                        | 24509           | 24509  | 24509  | 24509  | 91      |
| updateStaderOracle                               | 24443           | 24443  | 24443  | 24443  | 115     |
| updateStaderToken                                | 24521           | 24521  | 24521  | 24521  | 38      |
| updateStaderTreasury                             | 24532           | 25865  | 26532  | 26532  | 6       |
| updateStakePoolManager                           | 24489           | 24598  | 24489  | 26489  | 91      |
| updateValidatorWithdrawalVaultImplementation     | 24444           | 24444  | 24444  | 24444  | 10      |
| updateVaultFactory                               | 24466           | 24466  | 24466  | 24466  | 85      |
| updateWithdrawnKeysBatchSize                     | 2295            | 5745   | 5745   | 9195   | 4       |


| contracts/StaderInsuranceFund.sol:StaderInsuranceFund contract |                 |       |        |       |         |
|----------------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                                | Deployment Size |       |        |       |         |
| 856762                                                         | 4399            |       |        |       |         |
| Function Name                                                  | min             | avg   | median | max   | # calls |
| DEFAULT_ADMIN_ROLE                                             | 261             | 261   | 261    | 261   | 1       |
| depositFund                                                    | 1243            | 1243  | 1243   | 1243  | 3       |
| hasRole                                                        | 2675            | 2675  | 2675   | 2675  | 1       |
| initialize                                                     | 93842           | 93842 | 93842  | 93842 | 7       |
| reimburseUserFund                                              | 10953           | 16431 | 16431  | 21910 | 2       |
| staderConfig                                                   | 382             | 1382  | 1382   | 2382  | 2       |
| updateStaderConfig                                             | 8911            | 20569 | 20569  | 32227 | 2       |
| withdrawFund                                                   | 20533           | 35692 | 35692  | 50852 | 2       |


| contracts/ValidatorWithdrawalVault.sol:ValidatorWithdrawalVault contract |                 |        |        |        |         |
|--------------------------------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                                          | Deployment Size |        |        |        |         |
| 1144787                                                                  | 5750            |        |        |        |         |
| Function Name                                                            | min             | avg    | median | max    | # calls |
| calculateValidatorWithdrawalShare                                        | 10752           | 18111  | 21791  | 21791  | 3       |
| distributeRewards                                                        | 11501           | 81847  | 80231  | 155427 | 4       |
| receive                                                                  | 1491            | 1491   | 1491   | 1491   | 1       |
| settleFunds                                                              | 12402           | 117750 | 163205 | 177645 | 3       |


| contracts/VaultProxy.sol:VaultProxy contract |                 |        |        |        |         |
|----------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                              | Deployment Size |        |        |        |         |
| 371811                                       | 1889            |        |        |        |         |
| Function Name                                | min             | avg    | median | max    | # calls |
| calculateValidatorWithdrawalShare            | 21969           | 29328  | 33008  | 33008  | 3       |
| distributeRewards                            | 23891           | 95396  | 95571  | 166552 | 4       |
| fallback                                     | 21451           | 21458  | 21458  | 21465  | 2       |
| id                                           | 383             | 2216   | 2383   | 2383   | 12      |
| initialise                                   | 605             | 73740  | 71083  | 101983 | 104     |
| isInitialized                                | 2333            | 2333   | 2333   | 2333   | 2       |
| isValidatorWithdrawalVault                   | 399             | 399    | 399    | 399    | 2       |
| owner                                        | 381             | 1714   | 2381   | 2381   | 6       |
| poolId                                       | 369             | 369    | 369    | 369    | 15      |
| settleFunds                                  | 32885           | 126335 | 165840 | 180280 | 3       |
| staderConfig                                 | 404             | 639    | 404    | 2404   | 17      |
| updateOwner                                  | 2499            | 3572   | 3572   | 4645   | 4       |
| updateStaderConfig                           | 2543            | 5621   | 5621   | 8700   | 4       |
| withdraw                                     | 24247           | 80548  | 86120  | 131277 | 3       |


| contracts/factory/VaultFactory.sol:VaultFactory contract |                 |        |        |        |         |
|----------------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                          | Deployment Size |        |        |        |         |
| 1364689                                                  | 6936            |        |        |        |         |
| Function Name                                            | min             | avg    | median | max    | # calls |
| NODE_REGISTRY_CONTRACT                                   | 305             | 305    | 305    | 305    | 85      |
| computeNodeELRewardVaultAddress                          | 1621            | 1621   | 1621   | 1621   | 1       |
| deployNodeELRewardVault                                  | 115833          | 124300 | 126333 | 126333 | 31      |
| deployWithdrawVault                                      | 116015          | 117765 | 116015 | 126515 | 54      |
| grantRole                                                | 27549           | 27549  | 27549  | 27549  | 85      |
| initialize                                               | 497875          | 497875 | 497875 | 497875 | 85      |


| node_modules/@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol:TransparentUpgradeableProxy contract |                 |        |         |         |         |
|-----------------------------------------------------------------------------------------------------------------------------|-----------------|--------|---------|---------|---------|
| Deployment Cost                                                                                                             | Deployment Size |        |         |         |         |
| 465420                                                                                                                      | 3764            |        |         |         |         |
| Function Name                                                                                                               | min             | avg    | median  | max     | # calls |
| BURNER_ROLE                                                                                                                 | 1102            | 1102   | 1102    | 1102    | 2       |
| DEFAULT_ADMIN_ROLE                                                                                                          | 1056            | 1072   | 1057    | 1123    | 7       |
| MANAGER                                                                                                                     | 1159            | 1159   | 1159    | 1159    | 162     |
| MINTER_ROLE                                                                                                                 | 7556            | 7556   | 7556    | 7556    | 3       |
| MIN_AUCTION_DURATION                                                                                                        | 1056            | 5389   | 7556    | 7556    | 3       |
| NODE_REGISTRY_CONTRACT                                                                                                      | 1100            | 1100   | 1100    | 1100    | 85      |
| OPERATOR                                                                                                                    | 1080            | 1080   | 1080    | 1080    | 86      |
| PERMISSIONED_POOL                                                                                                           | 1125            | 1797   | 1125    | 7625    | 29      |
| PERMISSIONLESS_POOL                                                                                                         | 1080            | 3246   | 1080    | 7580    | 15      |
| POOL_ID                                                                                                                     | 1132            | 1132   | 1132    | 1132    | 1       |
| STADER_ORACLE                                                                                                               | 1103            | 1103   | 1103    | 1103    | 6       |
| activateNodeOperator                                                                                                        | 3524            | 14288  | 14288   | 25052   | 2       |
| addBid                                                                                                                      | 1277            | 43290  | 69972   | 69972   | 15      |
| addValidatorKeys                                                                                                            | 2756            | 899162 | 1445542 | 1445769 | 29      |
| allocateValidatorsAndUpdateOperatorId                                                                                       | 8800            | 10869  | 10911   | 12855   | 4       |
| balanceOf                                                                                                                   | 1425            | 2225   | 1425    | 3425    | 5       |
| balances                                                                                                                    | 1379            | 3267   | 1379    | 9879    | 9       |
| bidIncrement                                                                                                                | 1134            | 1251   | 1134    | 3134    | 17      |
| burnFrom                                                                                                                    | 1819            | 9634   | 4420    | 33093   | 5       |
| calculateMEVTheftPenalty                                                                                                    | 2491            | 7659   | 7659    | 12827   | 2       |
| calculateMissedAttestationPenalty                                                                                           | 25138           | 25138  | 25138   | 25138   | 1       |
| changeSocializingPoolState                                                                                                  | 1908            | 27762  | 6099    | 96943   | 4       |
| claim                                                                                                                       | 36324           | 36324  | 36324   | 36324   | 3       |
| claimSD                                                                                                                     | 1346            | 9407   | 1495    | 41225   | 5       |
| computeNodeELRewardVaultAddress                                                                                             | 2422            | 2422   | 2422    | 2422    | 1       |
| convertETHToSD                                                                                                              | 5220            | 14470  | 11720   | 29220   | 4       |
| convertSDToETH                                                                                                              | 5300            | 5300   | 5300    | 5300    | 1       |
| createLot                                                                                                                   | 62424           | 109621 | 117353  | 119353  | 16      |
| deactivateNodeOperator                                                                                                      | 3542            | 3933   | 4129    | 4129    | 3       |
| deployNodeELRewardVault                                                                                                     | 116634          | 130343 | 133634  | 133634  | 31      |
| deployWithdrawVault                                                                                                         | 116828          | 119661 | 116828  | 133828  | 54      |
| depositFor                                                                                                                  | 11859           | 24379  | 23259   | 31759   | 5       |
| depositFund                                                                                                                 | 8535            | 8535   | 8535    | 8535    | 3       |
| depositSDAsCollateral                                                                                                       | 46085           | 51691  | 51573   | 61733   | 7       |
| duration                                                                                                                    | 1136            | 1302   | 1136    | 3136    | 12      |
| extractNonBidSD                                                                                                             | 1281            | 7032   | 1407    | 29656   | 5       |
| getAdmin                                                                                                                    | 1290            | 2040   | 1290    | 9790    | 102     |
| getAllActiveValidators                                                                                                      | 1203            | 1931   | 1931    | 2659    | 4       |
| getAuctionContract                                                                                                          | 1301            | 1801   | 1301    | 3301    | 8       |
| getCollateralETH                                                                                                            | 7620            | 7620   | 7620    | 7620    | 2       |
| getDecimals                                                                                                                 | 1194            | 1971   | 1194    | 3194    | 18      |
| getETHxToken                                                                                                                | 9755            | 9755   | 9755    | 9755    | 1       |
| getMinimumSDToBond                                                                                                          | 12345           | 12345  | 12345   | 12345   | 1       |
| getNodeELRewardVaultImplementation                                                                                          | 3277            | 6527   | 6527    | 9777    | 4       |
| getNodeELVaultAddressForOptOutOperators                                                                                     | 1225            | 81268  | 81268   | 161312  | 2       |
| getOperatorRewardAddress                                                                                                    | 1350            | 1361   | 1361    | 1372    | 2       |
| getOperatorRewardsCollector                                                                                                 | 3279            | 3279   | 3279    | 3279    | 7       |
| getOperatorTotalKeys                                                                                                        | 1303            | 1314   | 1314    | 1326    | 4       |
| getOperatorTotalNonTerminalKeys                                                                                             | 1305            | 1569   | 1569    | 1833    | 4       |
| getOperatorWithdrawThreshold                                                                                                | 29473           | 29473  | 29473   | 29473   | 2       |
| getPenaltyContract                                                                                                          | 1234            | 2567   | 3234    | 3234    | 3       |
| getPermissionedPool                                                                                                         | 1300            | 2279   | 1300    | 3300    | 49      |
| getPermissionedSocializingPool                                                                                              | 1255            | 3159   | 3255    | 3255    | 21      |
| getPermissionlessPool                                                                                                       | 1278            | 2017   | 1278    | 3278    | 73      |
| getPermissionlessSocializingPool                                                                                            | 3269            | 3269   | 3269    | 3269    | 16      |
| getPoolSelector                                                                                                             | 3256            | 3256   | 3256    | 3256    | 1       |
| getPoolUtils                                                                                                                | 1280            | 4387   | 3280    | 9780    | 121     |
| getPreDepositSize                                                                                                           | 1264            | 3041   | 3264    | 3264    | 9       |
| getRemainingSDToBond                                                                                                        | 6401            | 9415   | 9415    | 12430   | 2       |
| getRewardEligibleSD                                                                                                         | 27596           | 27596  | 27596   | 27596   | 1       |
| getRewardsThreshold                                                                                                         | 1262            | 1762   | 1262    | 3262    | 4       |
| getSDCollateral                                                                                                             | 1235            | 3139   | 3235    | 3235    | 21      |
| getSocializingPoolOptInCoolingPeriod                                                                                        | 1196            | 1862   | 1196    | 3196    | 3       |
| getSocializingPoolStateChangeBlock                                                                                          | 1302            | 1302   | 1302    | 1302    | 3       |
| getStaderInsuranceFund                                                                                                      | 3256            | 3256   | 3256    | 3256    | 1       |
| getStaderOracle                                                                                                             | 1256            | 2732   | 1256    | 9756    | 21      |
| getStaderToken                                                                                                              | 1278            | 6492   | 9778    | 9778    | 28      |
| getStaderTreasury                                                                                                           | 1235            | 1485   | 1235    | 3235    | 8       |
| getStakePoolManager                                                                                                         | 1300            | 1550   | 1300    | 3300    | 8       |
| getStakedEthPerNode                                                                                                         | 1259            | 2415   | 1259    | 9759    | 16      |
| getTotalActiveValidatorCount                                                                                                | 1198            | 1198   | 1198    | 1198    | 2       |
| getTotalFee                                                                                                                 | 1217            | 1772   | 1217    | 3217    | 18      |
| getTotalQueuedValidatorCount                                                                                                | 3421            | 4202   | 4073    | 6935    | 7       |
| getValidatorWithdrawalVaultImplementation                                                                                   | 1302            | 3743   | 3302    | 9802    | 17      |
| getValidatorsByOperator                                                                                                     | 7779            | 13329  | 10289   | 21921   | 6       |
| getVaultFactory                                                                                                             | 1236            | 2623   | 3236    | 3236    | 49      |
| getWithdrawnKeyBatchSize                                                                                                    | 1295            | 1295   | 1295    | 1295    | 6       |
| grantRole                                                                                                                   | 28325           | 28408  | 28371   | 35271   | 339     |
| hasEnoughSDCollateral                                                                                                       | 6470            | 6470   | 6470    | 6470    | 1       |
| hasRole                                                                                                                     | 3476            | 3530   | 3543    | 3588    | 7       |
| increaseTotalActiveValidatorCount                                                                                           | 24221           | 29386  | 28103   | 39103   | 11      |
| initialize(address,address)                                                                                                 | 75963           | 297980 | 356721  | 498673  | 425     |
| initialize(address,address,address)                                                                                         | 184399          | 184399 | 184399  | 184399  | 12      |
| inputKeyCountLimit                                                                                                          | 1197            | 1208   | 1208    | 1219    | 4       |
| isExistingOperator                                                                                                          | 1462            | 4295   | 1462    | 9962    | 3       |
| isExistingPubkey                                                                                                            | 1720            | 1731   | 1731    | 1743    | 2       |
| lots                                                                                                                        | 2170            | 4323   | 4170    | 8170    | 13      |
| markValidatorReadyToDeposit                                                                                                 | 7786            | 56319  | 14727   | 168797  | 8       |
| markValidatorSettled                                                                                                        | 10354           | 10389  | 10389   | 10425   | 2       |
| markValidatorStatusAsPreDeposit                                                                                             | 27523           | 27523  | 27523   | 27523   | 6       |
| maxApproveSD                                                                                                                | 32122           | 32122  | 32122   | 32122   | 1       |
| maxNonTerminalKeyPerOperator                                                                                                | 1211            | 1233   | 1233    | 1256    | 4       |
| mevTheftPenaltyPerStrike                                                                                                    | 1179            | 2679   | 1179    | 9679    | 7       |
| mint                                                                                                                        | 1863            | 31012  | 40678   | 50219   | 6       |
| missedAttestationPenaltyPerStrike                                                                                           | 1135            | 2885   | 1135    | 9635    | 6       |
| nextLot                                                                                                                     | 1260            | 4447   | 1260    | 9760    | 8       |
| nextOperatorId                                                                                                              | 1159            | 2514   | 3159    | 3225    | 3       |
| nextQueuedValidatorIndex                                                                                                    | 1179            | 1179   | 1179    | 1179    | 1       |
| nextQueuedValidatorIndexByOperatorId                                                                                        | 1369            | 1369   | 1369    | 1369    | 2       |
| nextValidatorId                                                                                                             | 1178            | 1855   | 1200    | 3200    | 6       |
| nodeELRewardVaultByOperatorId                                                                                               | 1343            | 1343   | 1343    | 1343    | 4       |
| onboardNodeOperator(bool,string,address)(address)                                                                           | 1817            | 313545 | 346434  | 386956  | 34      |
| onboardNodeOperator(string,address)(address)                                                                                | 1753            | 165524 | 191287  | 234684  | 26      |
| onlyManagerRole                                                                                                             | 1613            | 5944   | 10113   | 10113   | 101     |
| onlyOperatorRole                                                                                                            | 1626            | 5356   | 3626    | 10126   | 26      |
| onlyPreDepositValidator                                                                                                     | 1912            | 1923   | 1923    | 1934    | 2       |
| onlyStaderContract                                                                                                          | 1504            | 1984   | 1504    | 3504    | 50      |
| operatorIDByAddress                                                                                                         | 1378            | 1395   | 1378    | 1422    | 15      |
| operatorIdForExcessDeposit                                                                                                  | 1202            | 1602   | 1202    | 3202    | 5       |
| operatorSDBalance                                                                                                           | 1421            | 1643   | 1421    | 3421    | 9       |
| operatorStructById                                                                                                          | 3126            | 3133   | 3126    | 3149    | 3       |
| pause                                                                                                                       | 16221           | 32457  | 37579   | 46171   | 5       |
| poolThresholdbyPoolId                                                                                                       | 2707            | 2707   | 2707    | 2707    | 1       |
| ratedOracleAddress                                                                                                          | 1178            | 4678   | 3178    | 9678    | 3       |
| reimburseUserFund                                                                                                           | 11752           | 17230  | 17230   | 22709   | 2       |
| setAdditionalPenaltyAmount                                                                                                  | 23064           | 23095  | 23095   | 23127   | 2       |
| slashValidatorSD                                                                                                            | 58052           | 96675  | 77817   | 170231  | 6       |
| socializingPoolStateChangeBlock                                                                                             | 1327            | 1334   | 1327    | 1349    | 3       |
| staderConfig                                                                                                                | 1177            | 5750   | 9677    | 9783    | 15      |
| totalActiveValidatorCount                                                                                                   | 1158            | 4571   | 1180    | 9680    | 5       |
| totalPenaltyAmount                                                                                                          | 1784            | 1784   | 1784    | 1784    | 3       |
| totalSupply                                                                                                                 | 3122            | 3122   | 3122    | 3122    | 1       |
| transferCollateralToPool                                                                                                    | 29318           | 33007  | 33007   | 36696   | 2       |
| transferHighestBidToSSPM                                                                                                    | 4608            | 13313  | 8442    | 38634   | 5       |
| unpause                                                                                                                     | 2284            | 10683  | 3760    | 32930   | 4       |
| updateAdmin                                                                                                                 | 28239           | 28239  | 28239   | 28239   | 16      |
| updateAuctionContract                                                                                                       | 25240           | 25240  | 25240   | 25240   | 19      |
| updateBidIncrement                                                                                                          | 28916           | 28916  | 28916   | 28916   | 1       |
| updateDepositStatusAndBlock                                                                                                 | 6545            | 20767  | 26453   | 26461   | 14      |
| updateDuration                                                                                                              | 16363           | 19375  | 19375   | 22387   | 2       |
| updateETHxToken                                                                                                             | 25316           | 25316  | 25316   | 25316   | 6       |
| updateInputKeyCountLimit                                                                                                    | 22889           | 24953  | 24953   | 27018   | 4       |
| updateMEVTheftPenaltyPerStrike                                                                                              | 20249           | 21525  | 21525   | 22801   | 2       |
| updateMaxNonTerminalKeyPerOperator                                                                                          | 22909           | 25668  | 27027   | 27056   | 6       |
| updateMaxOperatorId                                                                                                         | 3638            | 6398   | 6398    | 9158    | 2       |
| updateMissedAttestationPenaltyPerStrike                                                                                     | 20315           | 21591  | 21591   | 22867   | 2       |
| updateNextQueuedValidatorIndex                                                                                              | 47349           | 47349  | 47349   | 47349   | 1       |
| updateNodeELRewardImplementation                                                                                            | 25283           | 25283  | 25283   | 25283   | 91      |
| updateOperatorDetails                                                                                                       | 4471            | 11781  | 5702    | 31223   | 8       |
| updateOperatorRewardsCollector                                                                                              | 25284           | 25284  | 25284   | 25284   | 101     |
| updatePenaltyContract                                                                                                       | 25259           | 25259  | 25259   | 25259   | 106     |
| updatePermissionedPool                                                                                                      | 25239           | 25451  | 25239   | 33739   | 40      |
| updatePermissionedSocializingPool                                                                                           | 25238           | 25238  | 25238   | 25238   | 39      |
| updatePermissionlessPool                                                                                                    | 25283           | 25283  | 25283   | 25283   | 46      |
| updatePermissionlessSocializingPool                                                                                         | 25263           | 25263  | 25263   | 25263   | 46      |
| updatePoolThreshold                                                                                                         | 23391           | 84899  | 114086  | 114086  | 10      |
| updatePoolUtils                                                                                                             | 25262           | 25262  | 25262   | 25262   | 131     |
| updateQueuedValidatorIndex                                                                                                  | 26270           | 26795  | 26270   | 28370   | 4       |
| updateRatedOracleAddress                                                                                                    | 16443           | 19950  | 20532   | 22877   | 3       |
| updateRewardsThreshold                                                                                                      | 33656           | 33656  | 33656   | 33656   | 2       |
| updateSDCollateral                                                                                                          | 25285           | 25285  | 25285   | 25285   | 95      |
| updateSocializingPoolOptInCoolingPeriod                                                                                     | 33612           | 33612  | 33612   | 33612   | 2       |
| updateStaderConfig                                                                                                          | 7799            | 24948  | 16424   | 39721   | 14      |
| updateStaderInsuranceFund                                                                                                   | 25304           | 25304  | 25304   | 25304   | 91      |
| updateStaderOracle                                                                                                          | 25238           | 25238  | 25238   | 25238   | 115     |
| updateStaderToken                                                                                                           | 25316           | 25316  | 25316   | 25316   | 38      |
| updateStaderTreasury                                                                                                        | 25327           | 26660  | 27327   | 27327   | 6       |
| updateStakePoolManager                                                                                                      | 25284           | 25608  | 25284   | 33784   | 91      |
| updateTotalPenaltyAmount                                                                                                    | 22792           | 56246  | 54477   | 91469   | 3       |
| updateValidatorExitPenaltyThreshold                                                                                         | 20293           | 21569  | 21569   | 22845   | 2       |
| updateValidatorWithdrawalVaultImplementation                                                                                | 25239           | 25239  | 25239   | 25239   | 10      |
| updateVaultFactory                                                                                                          | 25261           | 25261  | 25261   | 25261   | 85      |
| updateVerifiedKeysBatchSize                                                                                                 | 3619            | 21098  | 25913   | 28952   | 8       |
| updateWithdrawnKeysBatchSize                                                                                                | 3090            | 9790   | 9790    | 16490   | 4       |
| validatorExitPenaltyThreshold                                                                                               | 1179            | 4679   | 3179    | 9679    | 3       |
| validatorIdByPubkey                                                                                                         | 1702            | 1702   | 1702    | 1702    | 24      |
| validatorRegistry                                                                                                           | 6395            | 6406   | 6406    | 6417    | 6       |
| verifiedKeyBatchSize                                                                                                        | 1179            | 2179   | 2179    | 3179    | 4       |
| whitelistPermissionedNOs                                                                                                    | 27279           | 31357  | 27279   | 46779   | 51      |
| withdraw                                                                                                                    | 12165           | 24134  | 19845   | 40393   | 3       |
| withdrawFund                                                                                                                | 21332           | 36489  | 36489   | 51647   | 2       |
| withdrawUnselectedBid                                                                                                       | 4727            | 11331  | 6378    | 32652   | 5       |
| withdrawnValidators                                                                                                         | 7664            | 17280  | 10191   | 37885   | 6       |


| test/mocks/NodeRegistryMock.sol:NodeRegistryMock contract |                 |       |        |       |         |
|-----------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                           | Deployment Size |       |        |       |         |
| 415742                                                    | 2084            |       |        |       |         |
| Function Name                                             | min             | avg   | median | max   | # calls |
| getOperatorTotalKeys                                      | 274             | 274   | 274    | 274   | 6       |
| operatorIDByAddress                                       | 364             | 364   | 364    | 364   | 9       |
| operatorStructById                                        | 1674            | 8874  | 9674   | 9674  | 10      |
| validatorRegistry                                         | 3790            | 14142 | 19790  | 19790 | 17      |


| test/mocks/OperatorRewardsCollectorMock.sol:OperatorRewardsCollectorMock contract |                 |     |        |     |         |
|-----------------------------------------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                                                   | Deployment Size |     |        |     |         |
| 29881                                                                             | 179             |     |        |     |         |
| Function Name                                                                     | min             | avg | median | max | # calls |
| depositFor                                                                        | 211             | 211 | 211    | 211 | 2       |


| test/mocks/PenaltyMockForVault.sol:PenaltyMockForVault contract |                 |       |        |       |         |
|-----------------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                                 | Deployment Size |       |        |       |         |
| 171814                                                          | 890             |       |        |       |         |
| Function Name                                                   | min             | avg   | median | max   | # calls |
| markValidatorSettled                                            | 246             | 246   | 246    | 246   | 2       |
| totalPenaltyAmount                                              | 0               | 409   | 409    | 819   | 2       |
| updateTotalPenaltyAmount                                        | 23076           | 23076 | 23076  | 23076 | 2       |


| test/mocks/PermissionedPoolMock.sol:PermissionedPoolMock contract |                 |     |        |     |         |
|-------------------------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                                   | Deployment Size |     |        |     |         |
| 154726                                                            | 840             |     |        |     |         |
| Function Name                                                     | min             | avg | median | max | # calls |
| fullDepositOnBeaconChain                                          | 397             | 397 | 397    | 397 | 1       |
| transferETHOfDefectiveKeysToSSPM                                  | 233             | 233 | 233    | 233 | 1       |


| test/mocks/PermissionlessPoolMock.sol:PermissionlessPoolMock contract |                 |     |        |     |         |
|-----------------------------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                                       | Deployment Size |     |        |     |         |
| 65317                                                                 | 358             |     |        |     |         |
| Function Name                                                         | min             | avg | median | max | # calls |
| preDepositOnBeaconChain                                               | 707             | 707 | 707    | 707 | 8       |
| receiveRemainingCollateralETH                                         | 74              | 74  | 74     | 74  | 1       |


| test/mocks/PoolUtilsMock.sol:PoolUtilsMock contract |                 |      |        |       |         |
|-----------------------------------------------------|-----------------|------|--------|-------|---------|
| Deployment Cost                                     | Deployment Size |      |        |       |         |
| 816251                                              | 3843            |      |        |       |         |
| Function Name                                       | min             | avg  | median | max   | # calls |
| calculateRewardShare                                | 6691            | 9579 | 10691  | 12691 | 9       |
| getCollateralETH                                    | 469             | 469  | 469    | 469   | 5       |
| getNodeRegistry                                     | 482             | 1482 | 1482   | 2482  | 26      |
| getOperatorPoolId                                   | 431             | 431  | 431    | 431   | 9       |
| getOperatorTotalNonTerminalKeys                     | 623             | 1956 | 2623   | 2623  | 6       |
| nodeRegistry                                        | 403             | 1403 | 1403   | 2403  | 6       |


| test/mocks/PoolUtilsMockForDepositFlow.sol:PoolUtilsMockForDepositFlow contract |                 |      |        |      |         |
|---------------------------------------------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                                                                 | Deployment Size |      |        |      |         |
| 320734                                                                          | 1633            |      |        |      |         |
| Function Name                                                                   | min             | avg  | median | max  | # calls |
| calculateRewardShare                                                            | 420             | 420  | 420    | 420  | 1       |
| getNodeRegistry                                                                 | 2505            | 2505 | 2505   | 2505 | 1       |
| isExistingOperator                                                              | 0               | 394  | 410    | 410  | 54      |
| onlyValidKeys                                                                   | 1051            | 1051 | 1051   | 1051 | 54      |
| onlyValidName                                                                   | 561             | 561  | 561    | 572  | 64      |
| poolAddressById                                                                 | 0               | 3751 | 4279   | 4302 | 58      |


| test/mocks/SDCollateralMock.sol:SDCollateralMock contract |                 |     |        |     |         |
|-----------------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                           | Deployment Size |     |        |     |         |
| 59711                                                     | 330             |     |        |     |         |
| Function Name                                             | min             | avg | median | max | # calls |
| hasEnoughSDCollateral                                     | 0               | 387 | 431    | 431 | 20      |
| slashValidatorSD                                          | 286             | 286 | 286    | 286 | 1       |


| test/mocks/StaderInsuranceFundMock.sol:StaderInsuranceFundMock contract |                 |     |        |     |         |
|-------------------------------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                                         | Deployment Size |     |        |     |         |
| 17869                                                                   | 118             |     |        |     |         |
| Function Name                                                           | min             | avg | median | max | # calls |
| depositFund                                                             | 74              | 74  | 74     | 74  | 1       |


| test/mocks/StaderOracleMock.sol:StaderOracleMock contract |                 |     |        |     |         |
|-----------------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                           | Deployment Size |     |        |     |         |
| 25075                                                     | 154             |     |        |     |         |
| Function Name                                             | min             | avg | median | max | # calls |
| getSDPriceInETH                                           | 146             | 146 | 146    | 146 | 18      |


| test/mocks/StaderTokenMock.sol:StaderTokenMock contract |                 |       |        |       |         |
|---------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                         | Deployment Size |       |        |       |         |
| 1658459                                                 | 10342           |       |        |       |         |
| Function Name                                           | min             | avg   | median | max   | # calls |
| approve                                                 | 24651           | 24651 | 24651  | 24651 | 21      |
| balanceOf                                               | 563             | 1763  | 2563   | 2563  | 30      |
| transfer                                                | 3793            | 22377 | 20555  | 34493 | 5       |
| transferFrom                                            | 1016            | 23617 | 28196  | 29796 | 23      |


| test/mocks/StakePoolManagerMock.sol:StakePoolManagerMock contract |                 |     |        |     |         |
|-------------------------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                                   | Deployment Size |     |        |     |         |
| 21875                                                             | 138             |     |        |     |         |
| Function Name                                                     | min             | avg | median | max | # calls |
| receiveEthFromAuction                                             | 118             | 118 | 118    | 118 | 1       |
| receiveExecutionLayerRewards                                      | 96              | 96  | 96     | 96  | 2       |
| receiveWithdrawVaultUserShare                                     | 74              | 74  | 74     | 74  | 4       |
```