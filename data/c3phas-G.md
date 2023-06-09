# Codebase optimization report

## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Table Of Contents

- [Codebase optimization report](#codebase-optimization-report)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Table Of Contents](#table-of-contents)
  - [Codebase impressions](#codebase-impressions)
  - [`uint256` to `bool` `mapping`: Utilizing Bitmaps to dramatically save on Gas](#uint256-to-bool-mapping-utilizing-bitmaps-to-dramatically-save-on-gas)
  - [Using `this.<fn>()` wastes gas](#using-thisfn-wastes-gas)
  - [Expensive operation inside a for loop](#expensive-operation-inside-a-for-loop)
    - [Cache `staderConfig` and `validatorExitPenaltyThreshold`  outside the loop](#cache-staderconfig-and-validatorexitpenaltythreshold--outside-the-loop)
    - [We can cache the value of `nextOperatorId` outside the loop](#we-can-cache-the-value-of-nextoperatorid-outside-the-loop)
    - [We can cache the value of `nextOperatorId` outside the loop](#we-can-cache-the-value-of-nextoperatorid-outside-the-loop-1)
    - [Cache `poolAllocationMaxSize` outside the loop](#cache-poolallocationmaxsize-outside-the-loop)
    - [Looping on state variables is too expensive(cache `nextRequestIdToFinalize` outside the loop)](#looping-on-state-variables-is-too-expensivecache-nextrequestidtofinalize-outside-the-loop)
  - [External function calls inside a loop are too expensive](#external-function-calls-inside-a-loop-are-too-expensive)
    - [Don't make function calls inside a loop(Here we call `staderConfig.getStakedEthPerNode()`)](#dont-make-function-calls-inside-a-loophere-we-call-staderconfiggetstakedethpernode)
    - [An external call `staderConfig.getFullDepositSize()` inside the for loop is too expensive](#an-external-call-staderconfiggetfulldepositsize-inside-the-for-loop-is-too-expensive)
    - [Calls to external functions should be done outside the loop](#calls-to-external-functions-should-be-done-outside-the-loop)
  - [Shortcircuit rules can be be used to our advantage. We can save ~2100 for the SLOAD incase of a revert on the check `address(this).balance == 0`](#shortcircuit-rules-can-be-be-used-to-our-advantage-we-can-save-2100-for-the-sload-incase-of-a-revert-on-the-check-addressthisbalance--0)
  - [Emitting storage values instead of the memory one.](#emitting-storage-values-instead-of-the-memory-one)
    - [Emitting storage values when we can emit a cheap operation wastes a lot of gas](#emitting-storage-values-when-we-can-emit-a-cheap-operation-wastes-a-lot-of-gas)
    - [Cached nextLot and changed other emit values](#cached-nextlot-and-changed-other-emit-values)
    - [Emit `_duration` instead duration](#emit-_duration-instead-duration)
    - [Emit `_maxNonTerminalKeyPerOperator` instead of maxNonTerminalKeyPerOperator](#emit-_maxnonterminalkeyperoperator-instead-of-maxnonterminalkeyperoperator)
    - [Emit `_inputKeyCountLimit` instead of `inputKeyCountLimit`](#emit-_inputkeycountlimit-instead-of-inputkeycountlimit)
    - [Emit `_erChangeLimit` instead of `erChangeLimit`](#emit-_erchangelimit-instead-of-erchangelimit)
    - [AS `erInspectionMode` is being set to `true` emitting `true` would avoid reading from state again](#as-erinspectionmode-is-being-set-to-true-emitting-true-would-avoid-reading-from-state-again)
    - [We have local variables equal to the state variables being emitted](#we-have-local-variables-equal-to-the-state-variables-being-emitted)
    - [Emit `_owner` instead of `owner`](#emit-_owner-instead-of-owner)
    - [Emit `_vaultProxyImpl` instead of `vaultProxyImplementation`](#emit-_vaultproxyimpl-instead-of-vaultproxyimplementation)
    - [Emit `_nextQueuedValidatorIndex` instead of `nextQueuedValidatorIndex`](#emit-_nextqueuedvalidatorindex-instead-of-nextqueuedvalidatorindex)
    - [Emit `_inputKeyCountLimit` instead of `inputKeyCountLimit`](#emit-_inputkeycountlimit-instead-of-inputkeycountlimit-1)
    - [Emit `_maxNonTerminalKeyPerOperator` instead of `maxNonTerminalKeyPerOperator`](#emit-_maxnonterminalkeyperoperator-instead-of-maxnonterminalkeyperoperator-1)
  - [Take advantage of cached values to avoid reading from storage](#take-advantage-of-cached-values-to-avoid-reading-from-storage)
    - [Avoid reading state if we have an equivalent memory value](#avoid-reading-state-if-we-have-an-equivalent-memory-value)
    - [If a global/local variable is equal to a storage value, we can use the global/local one](#if-a-globallocal-variable-is-equal-to-a-storage-value-we-can-use-the-globallocal-one)
  - [Using storage instead of memory for structs/arrays saves gas](#using-storage-instead-of-memory-for-structsarrays-saves-gas)
    - [We can use storage and then cache the values being used repeatedly](#we-can-use-storage-and-then-cache-the-values-being-used-repeatedly)
  - [Leverage mapping and dot notation for struct assignment](#leverage-mapping-and-dot-notation-for-struct-assignment)
  - [Caching to local storage can be used for assignment not just reading values.](#caching-to-local-storage-can-be-used-for-assignment-not-just-reading-values)
  - [Use calldata instead of memory for function parameters](#use-calldata-instead-of-memory-for-function-parameters)
  - [Avoiding Unnecessary SLOAD in Solidity Subtraction Operations](#avoiding-unnecessary-sload-in-solidity-subtraction-operations)
    - [Use the cached value: `operatorSDBalance[operator]` has been cached already (save 1 SLOAD ~100 gas)](#use-the-cached-value-operatorsdbalanceoperator-has-been-cached-already-save-1-sload-100-gas)
    - [`operatorSDBalance[_operator]` has been cached into `sdBalance` and the cached value should be used](#operatorsdbalance_operator-has-been-cached-into-sdbalance-and-the-cached-value-should-be-used)
  - [We can save 1 SLOAD plus a few gas cost for the subtractions](#we-can-save-1-sload-plus-a-few-gas-cost-for-the-subtractions)
  - [We can save SLOAD here and some small operation costs(~180 gas)](#we-can-save-sload-here-and-some-small-operation-costs180-gas)
  - [Optimizing check order for cost efficient function execution](#optimizing-check-order-for-cost-efficient-function-execution)
    - [Validate all values before reading storage](#validate-all-values-before-reading-storage)
    - [Reorder the checks to have the one that doesn't read from storage come first](#reorder-the-checks-to-have-the-one-that-doesnt-read-from-storage-come-first)
    - [Reorder the checks to have the one not reading from storage come first](#reorder-the-checks-to-have-the-one-not-reading-from-storage-come-first)
    - [Should we re implement the lib or moving is enough](#should-we-re-implement-the-lib-or-moving-is-enough)
    - [External calls are expensive compared to the nonzero checks](#external-calls-are-expensive-compared-to-the-nonzero-checks)
    - [As the first check reads from storage, it should be done after cheaper checks](#as-the-first-check-reads-from-storage-it-should-be-done-after-cheaper-checks)
    - [Cheaper checks should be done first](#cheaper-checks-should-be-done-first)
    - [Don't read any state if we might revert on function parameters and constants](#dont-read-any-state-if-we-might-revert-on-function-parameters-and-constants)
    - [The second check does not read from state thus should be done first](#the-second-check-does-not-read-from-state-thus-should-be-done-first)
    - [Similar to the above](#similar-to-the-above)
    - [The first checks reads state while the second doesn't , reorder them](#the-first-checks-reads-state-while-the-second-doesnt--reorder-them)
    - [checks for function params should come first](#checks-for-function-params-should-come-first)
    - [The second check is cheaper and should be done first](#the-second-check-is-cheaper-and-should-be-done-first)
    - [Do sanity checks immediately, Can verify `totalRewards` before making external calls](#do-sanity-checks-immediately-can-verify-totalrewards-before-making-external-calls)
    - [Validate all values before doing any external calls](#validate-all-values-before-doing-any-external-calls)
    - [Check functional params first being doing any external calls](#check-functional-params-first-being-doing-any-external-calls)
    - [Move the NonZero check to the top](#move-the-nonzero-check-to-the-top)
  - [The modifier can be rewritten to save some gas](#the-modifier-can-be-rewritten-to-save-some-gas)
  - [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
  - [Don't cache function params (save ~5 gas)](#dont-cache-function-params-save-5-gas)
  - [Short hand if](#short-hand-if)
  - [Caching calldata increases gas instead of saving it(Saves ~5 gas)](#caching-calldata-increases-gas-instead-of-saving-itsaves-5-gas)
  - [Caching global variables/ `msg.sender` increases gas](#caching-global-variables-msgsender-increases-gas)
  - [Conclusion](#conclusion)


## Codebase impressions

The codebase has a lot of improvements that can be done. Starting with the most obvious findings such as caching storage variables, caching function calls to packing variables.

The contracts in scope seems to be doing some very gas expensive operations which can make the project to expensive for users. Notable ones are state reads inside for loops, external calls inside for loops, unnecessary state reads etc.

The c4 bot did an amazing job on this one and identified majority of this issues. (Please read the bot report and consider implementing the suggested changes)
However, we have identified additional areas where optimization is possible, some of which may not be immediately apparent.


## `uint256` to `bool` `mapping`: Utilizing Bitmaps to dramatically save on Gas

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#L29
```solidity
File: /contracts/SocializingPool.sol
29:    mapping(address => mapping(uint256 => bool)) public override claimedRewards;
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#L30
```solidity
File: /contracts/SocializingPool.sol
30:    mapping(uint256 => bool) public handledRewards;
```

See https://soliditydeveloper.com/bitmaps



## Using `this.<fn>()` wastes gas

Calling an external function internally, through the use of `this` wastes the gas overhead of calling an external function (100 gas). Instead, change the function from `external to public`, and remove the `this`

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessPool.sol#L102-L107
```solidity
File: /contracts/PermissionlessPool.sol
102:            bytes32 depositDataRoot = this.computeDepositDataRoot(
103:                _pubkey[i],
104:                _preDepositSignature[i],
105:                withdrawCredential,
106:                staderConfig.getPreDepositSize()
107:            );
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessPool.sol#L256-L261
```solidity
File: /contracts/PermissionlessPool.sol
256:        bytes32 depositDataRoot = this.computeDepositDataRoot(
257:            pubkey,
258:            depositSignature,
259:            withdrawCredential,
260:            _DEPOSIT_SIZE
261:        );
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L147-L152
```solidity
File: /contracts/PermissionedPool.sol
147:            bytes32 depositDataRoot = this.computeDepositDataRoot(
148:                _pubkey[i],
149:                depositSignature,
150:                withdrawCredential,
151:                fullDepositSize
152:            );
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L302-L307
```solidity
File: /contracts/PermissionedPool.sol
302:        bytes32 depositDataRoot = this.computeDepositDataRoot(
303:            pubkey,
304:            preDepositSignature,
305:            withdrawCredential,
306:            preDepositSize
307:        );
```


## Expensive operation inside a for loop

**Note: The bot failed to capture state reads inside the loops**

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Penalty.sol#L98-L120

### Cache `staderConfig` and `validatorExitPenaltyThreshold`  outside the loop

```solidity
File: /contracts/Penalty.sol
98:    function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
99:        uint256 reportedValidatorCount = _pubkey.length;
100:        for (uint256 i; i < reportedValidatorCount; ) {
101:            if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
102:                revert ValidatorSettled();
103:            }

113:            if (totalPenalty >= validatorExitPenaltyThreshold) {
114:                emit ForceExitValidator(_pubkey[i]);
115:            }
```
`staderConfig` and `validatorExitPenaltyThreshold` are just simple storage reads therefore we can cache them outside the loop

```diff
     function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external overri
de nonReentrant {
         uint256 reportedValidatorCount = _pubkey.length;
+        uint256 _validatorExitPenaltyThreshold = validatorExitPenaltyThreshold;
+        IStaderConfig  _staderConfig = staderConfig;
         for (uint256 i; i < reportedValidatorCount; ) {
-            if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
+            if (UtilLib.getValidatorSettleStatus(_pubkey[i], _staderConfig)) {
                 revert ValidatorSettled();
             }
             bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
@@ -110,7 +112,7 @@ contract Penalty is IPenalty, Initializable, AccessControlUpgradeable, Reentranc
             // taking into account additional penalties and penalty reversals from the DAO.
             uint256 totalPenalty = _mevTheftPenalty + _missedAttestationPenalty + additionalPenaltyAmount[pubkeyRoot];
             totalPenaltyAmount[_pubkey[i]] = totalPenalty;
-            if (totalPenalty >= validatorExitPenaltyThreshold) {
+            if (totalPenalty >= _validatorExitPenaltyThreshold) {
                 emit ForceExitValidator(_pubkey[i]);
             }
             unchecked {
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L206-L217

### We can cache the value of `nextOperatorId` outside the loop

```solidity
File: /contracts/PermissionedNodeRegistry.sol
206:            for (uint256 i = 1; i < nextOperatorId; ) {
207:                if (!operatorStructById[i].active) {
208:                    continue;
209:                }
```

```diff
     function updateBidIncrement(uint256 _bidIncrement) external override {
diff --git a/contracts/PermissionedNodeRegistry.sol b/contracts/PermissionedNodeRegistry.sol
index ca8aa81..c2e3482 100644
--- a/contracts/PermissionedNodeRegistry.sol
+++ b/contracts/PermissionedNodeRegistry.sol
@@ -203,7 +203,8 @@ contract PermissionedNodeRegistry is
         uint256 totalValidatorToDeposit;
         bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);
         if (validatorPerOperatorGreaterThanZero) {
-            for (uint256 i = 1; i < nextOperatorId; ) {
+            uint256 _nextOperatorId = nextOperatorId;
+            for (uint256 i = 1; i < _nextOperatorId; ) {
                 if (!operatorStructById[i].active) {
                     continue;
                 }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L488

### We can cache the value of `nextOperatorId` outside the loop

```solidity
File: /contracts/PermissionedNodeRegistry.sol
488:        for (uint256 i = 1; i < nextOperatorId; ) {
```

```diff
     function getTotalQueuedValidatorCount() external view override returns (uin
t256) {
         uint256 totalQueuedValidators;
-        for (uint256 i = 1; i < nextOperatorId; ) {
+        uint256 _nextOperatorId = nextOperatorId;
+        for (uint256 i = 1; i < _nextOperatorId; ) {
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L90-L97

### Cache `poolAllocationMaxSize` outside the loop

```solidity
File: /contracts/PoolSelector.sol
90:        for (uint256 j; j < poolCount; ) {

94:            selectedPoolCapacity[i] = Math.min(
95:                poolAllocationMaxSize - selectedValidatorCount,
96:                Math.min(poolCapacity, remainingValidatorsToDeposit)
97:            );
```

```diff
diff --git a/contracts/PoolSelector.sol b/contracts/PoolSelector.sol
index bdb1db9..ac92961 100644
--- a/contracts/PoolSelector.sol
+++ b/contracts/PoolSelector.sol
@@ -87,12 +87,13 @@ contract PoolSelector is IPoolSelector, Initializable, AccessControlUpgradeable
         selectedPoolCapacity = new uint256[](poolCount);
         uint256 selectedValidatorCount;
         uint256 i = poolIdArrayIndexForExcessDeposit;
+        uint16 _poolAllocationMaxSize = poolAllocationMaxSize;
         for (uint256 j; j < poolCount; ) {
             uint256 poolCapacity = poolUtils.getQueuedValidatorCountByPool(poolIdArray[i]);
             uint256 poolDepositSize = ETH_PER_NODE - poolUtils.getCollateralETH(poolIdArray[i]);
             uint256 remainingValidatorsToDeposit = ethToDeposit / poolDepositSize;
             selectedPoolCapacity[i] = Math.min(
-                poolAllocationMaxSize - selectedValidatorCount,
+                _poolAllocationMaxSize - selectedValidatorCount,
                 Math.min(poolCapacity, remainingValidatorsToDeposit)
             );
             selectedValidatorCount += selectedPoolCapacity[i];
@@ -100,7 +101,7 @@ contract PoolSelector is IPoolSelector, Initializable, AccessControlUpgradeable
             i = (i + 1) % poolCount;
             //For ethToDeposit < ETH_PER_NODE, we will be able to at best deposit one more validator
             //but that will introduce complex logic, hence we are not solving that
-            if (ethToDeposit < ETH_PER_NODE || selectedValidatorCount >= poolAllocationMaxSize) {
+            if (ethToDeposit < ETH_PER_NODE || selectedValidatorCount >= _poolAllocationMaxSize) {
                 poolIdArrayIndexForExcessDeposit = i;
                 break;
             }
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/UserWithdrawalManager.sol#L132-L151

### Looping on state variables is too expensive(cache `nextRequestIdToFinalize` outside the loop)

```solidity
File: /contracts/UserWithdrawalManager.sol
132:        for (requestId = nextRequestIdToFinalize; requestId <= maxRequestIdToFinalize; ) {

151:        }
```

```diff
diff --git a/contracts/UserWithdrawalManager.sol b/contracts/UserWithdrawalManager.sol
index f563434..154f4fd 100644
--- a/contracts/UserWithdrawalManager.sol
+++ b/contracts/UserWithdrawalManager.sol
@@ -129,7 +129,8 @@ contract UserWithdrawalManager is
         uint256 ethToSendToFinalizeRequest;
         uint256 requestId;
         uint256 pooledETH = poolManager.balance;
-        for (requestId = nextRequestIdToFinalize; requestId <= maxRequestIdToFinalize; ) {
+        uint256 _nextRequestIdToFinalize = nextRequestIdToFinalize;
+        for (requestId = _nextRequestIdToFinalize; requestId <= maxRequestIdToFinalize; ) {
             UserWithdrawInfo memory userWithdrawInfo = userWithdrawRequests[requestId];
             uint256 requiredEth = userWithdrawInfo.ethExpected;
             uint256 lockedEthX = userWithdrawInfo.ethXAmount;
@@ -150,7 +151,7 @@ contract UserWithdrawalManager is
             }
         }
         // at here, upto (requestId-1) is finalized
-        if (requestId > nextRequestIdToFinalize) {
+        if (requestId > _nextRequestIdToFinalize) {
             nextRequestIdToFinalize = requestId;
             ETHx(staderConfig.getETHxToken()).burnFrom(address(this), lockedEthXToBurn);
             IStaderStakePoolManager(poolManager).transferETHToUserWithdrawManager(ethToSendToFinalizeRequest);
```

## External function calls inside a loop are too expensive

**Note: The bot only capture the second calls, but didn't capture the calls inside the loops**
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderStakePoolsManager.sol#L232-L239

### Don't make function calls inside a loop(Here we call `staderConfig.getStakedEthPerNode()`)

```solidity
File: /contracts/StaderStakePoolsManager.sol
232:        for (uint256 i = 0; i < poolCount; i++) {

238:            uint256 poolDepositSize = staderConfig.getStakedEthPerNode() -
239:                IPoolUtils(poolUtils).getCollateralETH(poolIdArray[i]);
```

```diff
diff --git a/contracts/StaderStakePoolsManager.sol b/contracts/StaderStakePoolsManager.sol
index c4bb7d5..deff6ed 100644
--- a/contracts/StaderStakePoolsManager.sol
+++ b/contracts/StaderStakePoolsManager.sol
@@ -229,13 +229,14 @@ contract StaderStakePoolsManager is
         ).poolAllocationForExcessETHDeposit(availableETHForNewDeposit);

         uint256 poolCount = poolIdArray.length;
+        uint256 _stakedEthPerNode = staderConfig.getStakedEthPerNode();
         for (uint256 i = 0; i < poolCount; i++) {
             uint256 validatorToDeposit = selectedPoolCapacity[i];
             if (validatorToDeposit == 0) {
                 continue;
             }
             address poolAddress = IPoolUtils(poolUtils).poolAddressById(poolIdArray[i]);
-            uint256 poolDepositSize = staderConfig.getStakedEthPerNode() -
+            uint256 poolDepositSize = _stakedEthPerNode -
                 IPoolUtils(poolUtils).getCollateralETH(poolIdArray[i]);

             lastExcessETHDepositBlock = block.number;
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessPool.sol#L137-L145

### An external call `staderConfig.getFullDepositSize()` inside the for loop is too expensive

```solidity
File: /contracts/PermissionlessPool.sol
137:        for (uint256 i = depositQueueStartIndex; i < requiredValidators + depositQueueStartIndex; ) {
138:            uint256 validatorId = IPermissionlessNodeRegistry(nodeRegistryAddress).queuedValidators(i);
139:            fullDepositOnBeaconChain(
140:                nodeRegistryAddress,
141:                vaultFactoryAddress,
142:                ethDepositContract,
143:                validatorId,
144:                staderConfig.getFullDepositSize()
145:            );
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessPool.sol#L94-L120

### Calls to external functions should be done outside the loop

```solidity
File: c/contracts/PermissionlessPool.sol
94:        for (uint256 i; i < pubkeyCount; ) {
95:            address withdrawVault = IVaultFactory(vaultFactory).computeWithdrawVaultAddress(
                96:INodeRegistry((staderConfig).getPermissionlessNodeRegistry()).POOL_ID(),
97:                _operatorId,
98:                _operatorTotalKeys + i
99:            );
100:            bytes memory withdrawCredential = IVaultFactory(vaultFactory).getValidatorWithdrawCredential(withdrawVault);

102:            bytes32 depositDataRoot = this.computeDepositDataRoot(
103:                _pubkey[i],
104:                _preDepositSignature[i],
105:                withdrawCredential,
106:                staderConfig.getPreDepositSize()
107:            );
108:            //slither-disable-next-line arbitrary-send-eth
            109:IDepositContract(staderConfig.getETHDepositContract()).deposit{value: staderConfig.getPreDepositSize()}(
110:                _pubkey[i],
111:                withdrawCredential,
112:                _preDepositSignature[i],
113:                depositDataRoot
114:            );
```

Calls to `staderConfig.getETHDepositContract()`,`staderConfig.getPreDepositSize()` and `staderConfig).getPermissionlessNodeRegistry()` should be made outside the loop and their results cached in memory


## Shortcircuit rules can be be used to our advantage. We can save ~2100 for the SLOAD incase of a revert on the check `address(this).balance == 0`

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L173-L180
```solidity
File: contracts/PermissionedPool.sol
173:    function transferExcessETHToSSPM() external nonReentrant {
174:        if (preDepositValidatorCount != 0 || address(this).balance == 0) {
175:            revert CouldNotDetermineExcessETH();
176        }
```
The operators `||` and `&&` apply the common short-circuiting rules. This means that in the expression “f(x) || g(y)”, if “f(x)” evaluates to true, “g(y)” will not be evaluated even if it may have side-effects.

This means that, if `preDepositValidatorCount != 0` we would not check the other condition  which is way cheaper(~100 gas)
Reading `preDepositValidatorCount` would costs us around 2100 gas as it's the first sload(COLD). We can rewrite the checks, to have the second one `address(this).balance == 0` come first. This way if the check returns true we would not need to evaluate the more expensive one.
```diff
     function transferExcessETHToSSPM() external nonReentrant {
-        if (preDepositValidatorCount != 0 || address(this).balance == 0) {
+        if (address(this).balance == 0 || preDepositValidatorCount != 0 ) {
             revert CouldNotDetermineExcessETH();
         }

```

## Emitting storage values instead of the memory one.

Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L29-L46

### Emitting storage values when we can emit a cheap operation wastes a lot of gas

```solidity
File: /contracts/Auction.sol
29:    function initialize(address _admin, address _staderConfig) external initializer {

38:        duration = 2 * MIN_AUCTION_DURATION;
39:        bidIncrement = 5e15;

44:        emit AuctionDurationUpdated(duration);
45:        emit BidIncrementUpdated(bidIncrement);
46:    }
```

```diff
diff --git a/contracts/Auction.sol b/contracts/Auction.sol
index ae5fd02..f14be8f 100644
--- a/contracts/Auction.sol
+++ b/contracts/Auction.sol
@@ -41,8 +41,8 @@ contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableU

         _grantRole(DEFAULT_ADMIN_ROLE, _admin);
         emit UpdatedStaderConfig(_staderConfig);
-        emit AuctionDurationUpdated(duration);
-        emit BidIncrementUpdated(bidIncrement);
+        emit AuctionDurationUpdated( 2 * MIN_AUCTION_DURATION);
+        emit BidIncrementUpdated(5e15);
     }
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L48-L60

### Cached nextLot and changed other emit values

```solidity
File: /contracts/Auction.sol
48:    function createLot(uint256 _sdAmount) external override whenNotPaused {
49:        lots[nextLot].startBlock = block.number;
50:        lots[nextLot].endBlock = block.number + duration;
51:        lots[nextLot].sdAmount = _sdAmount;

53:        LotItem storage lotItem = lots[nextLot];

55:        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
56:            revert SDTransferFailed();
57:        }
58:        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
59:        nextLot++;
60:    }
```

```diff
diff --git a/contracts/Auction.sol b/contracts/Auction.sol
index ae5fd02..1848658 100644
--- a/contracts/Auction.sol
+++ b/contracts/Auction.sol
@@ -46,16 +46,17 @@ contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableU
     }

     function createLot(uint256 _sdAmount) external override whenNotPaused {
-        lots[nextLot].startBlock = block.number;
-        lots[nextLot].endBlock = block.number + duration;
-        lots[nextLot].sdAmount = _sdAmount;
+        uint _nextLot = nextLot;
+        lots[_nextLot].startBlock = block.number;
+        lots[_nextLot].endBlock = block.number + duration;
+        lots[_nextLot].sdAmount = _sdAmount;

-        LotItem storage lotItem = lots[nextLot];
+        LotItem storage lotItem = lots[_nextLot];

         if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
             revert SDTransferFailed();
         }
-        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
+        emit LotCreated(_nextLot, _sdAmount, block.number, block.number + duration, bidIncrement);
         nextLot++;
     }
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L144-L149

### Emit `_duration` instead duration

```solidity
File: /contracts/Auction.sol
144:    function updateDuration(uint256 _duration) external override {
    
146:        duration = _duration;
147:        emit AuctionDurationUpdated(duration);
148:    }
```

```diff
diff --git a/contracts/Auction.sol b/contracts/Auction.sol
index ae5fd02..f2981cf 100644
--- a/contracts/Auction.sol
+++ b/contracts/Auction.sol
@@ -145,7 +145,7 @@ contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableU
         UtilLib.onlyManagerRole(msg.sender, staderConfig);
         if (_duration < MIN_AUCTION_DURATION) revert ShortDuration();
         duration = _duration;
-        emit AuctionDurationUpdated(duration);
+        emit AuctionDurationUpdated(_duration);
     }
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L416-L420

### Emit `_maxNonTerminalKeyPerOperator` instead of maxNonTerminalKeyPerOperator

```solidity
File: /contracts/PermissionedNodeRegistry.sol
416:    function updateMaxNonTerminalKeyPerOperator(uint64 _maxNonTerminalKeyPerOperator) external override {

417:        maxNonTerminalKeyPerOperator = _maxNonTerminalKeyPerOperator;
418:        emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);
419:    }
```

```diff
     function updateMaxNonTerminalKeyPerOperator(uint64 _maxNonTerminalKeyPerOperator) external override {
         UtilLib.onlyManagerRole(msg.sender, staderConfig);
         maxNonTerminalKeyPerOperator = _maxNonTerminalKeyPerOperator;
-        emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);
+        emit UpdatedMaxNonTerminalKeyPerOperator(_maxNonTerminalKeyPerOperator);
     }
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L427-L431

### Emit `_inputKeyCountLimit` instead of `inputKeyCountLimit`

```solidity
File: /contracts/PermissionedNodeRegistry.sol
427:    function updateInputKeyCountLimit(uint16 _inputKeyCountLimit) external override {
428:        UtilLib.onlyOperatorRole(msg.sender, staderConfig);
429:        inputKeyCountLimit = _inputKeyCountLimit;
430:        emit UpdatedInputKeyCountLimit(inputKeyCountLimit);
431:    }
```

```diff
     function updateInputKeyCountLimit(uint16 _inputKeyCountLimit) external override {
         UtilLib.onlyOperatorRole(msg.sender, staderConfig);
         inputKeyCountLimit = _inputKeyCountLimit;
-        emit UpdatedInputKeyCountLimit(inputKeyCountLimit);
+        emit UpdatedInputKeyCountLimit(_inputKeyCountLimit);
     }
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L530-L534

### Emit `_erChangeLimit` instead of `erChangeLimit`

```solidity
File: /contracts/StaderOracle.sol
530:    function updateERChangeLimit(uint256 _erChangeLimit) external override {

535:        erChangeLimit = _erChangeLimit;
536:        emit UpdatedERChangeLimit(erChangeLimit);
537:    }
```

```diff
diff --git a/contracts/StaderOracle.sol b/contracts/StaderOracle.sol
index ba007c5..31f948a 100644
--- a/contracts/StaderOracle.sol
+++ b/contracts/StaderOracle.sol
@@ -533,7 +533,7 @@ contract StaderOracle is IStaderOracle, AccessControlUpgradeable, PausableUpgrad
             revert ERPermissibleChangeOutofBounds();
         }
         erChangeLimit = _erChangeLimit;
-        emit UpdatedERChangeLimit(erChangeLimit);
+        emit UpdatedERChangeLimit(_erChangeLimit);
     }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L668-L673

### AS `erInspectionMode` is being set to `true` emitting `true` would avoid reading from state again

```solidity
File: /contracts/StaderOracle.sol
668:            erInspectionMode = true;

673:            emit ERInspectionModeActivated(erInspectionMode, block.timestamp);
```

```diff
diff --git a/contracts/StaderOracle.sol b/contracts/StaderOracle.sol
index ba007c5..35edf4d 100644
--- a/contracts/StaderOracle.sol
+++ b/contracts/StaderOracle.sol
@@ -670,7 +670,7 @@ contract StaderOracle is IStaderOracle, AccessControlUpgradeable, PausableUpgrad
             inspectionModeExchangeRate.totalETHBalance = _newTotalETHBalance;
             inspectionModeExchangeRate.totalETHXSupply = _newTotalETHXSupply;
             inspectionModeExchangeRate.reportingBlockNumber = _reportingBlockNumber;
-            emit ERInspectionModeActivated(erInspectionMode, block.timestamp);
+            emit ERInspectionModeActivated(true, block.timestamp);
             return;
         }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L684-L695

### We have local variables equal to the state variables being emitted

```solidity
File: /contracts/StaderOracle.sol
684:        exchangeRate.totalETHBalance = _totalETHBalance;
685:        exchangeRate.totalETHXSupply = _totalETHXSupply;
686:        exchangeRate.reportingBlockNumber = _reportingBlockNumber;

688:        // Emit balances updated event
689:        emit ExchangeRateUpdated(
690:            exchangeRate.reportingBlockNumber,
691:            exchangeRate.totalETHBalance,
692:            exchangeRate.totalETHXSupply,
693:            block.timestamp
694:        );
```

```diff
diff --git a/contracts/StaderOracle.sol b/contracts/StaderOracle.sol
index ba007c5..9355f75 100644
--- a/contracts/StaderOracle.sol
+++ b/contracts/StaderOracle.sol
@@ -687,9 +687,9 @@ contract StaderOracle is IStaderOracle, AccessControlUpgradeable, PausableUpgrad

         // Emit balances updated event
         emit ExchangeRateUpdated(
-            exchangeRate.reportingBlockNumber,
-            exchangeRate.totalETHBalance,
-            exchangeRate.totalETHXSupply,
+            _reportingBlockNumber,
+            _totalETHBalance,
+            _totalETHXSupply,
             block.timestamp
         );
     }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L68-L72

### Emit `_owner` instead of `owner`

```solidity
File: /contracts/VaultProxy.sol
68:    function updateOwner(address _owner) external override onlyOwner {
69:        UtilLib.checkNonZeroAddress(_owner);
70:        owner = _owner;
71:        emit UpdatedOwner(owner);
72:    }
```

```diff
     function updateOwner(address _owner) external override onlyOwner {
         UtilLib.checkNonZeroAddress(_owner);
         owner = _owner;
-        emit UpdatedOwner(owner);
+        emit UpdatedOwner(_owner);
     }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/factory/VaultFactory.sol#L95-L96

### Emit `_vaultProxyImpl` instead of `vaultProxyImplementation`

```solidity
File: /contracts/factory/VaultFactory.sol
95:        vaultProxyImplementation = _vaultProxyImpl;
96:        emit UpdatedVaultProxyImplementation(vaultProxyImplementation);
```

```diff
     function updateVaultProxyAddress(address _vaultProxyImpl) external override onlyRole(DEFAULT_ADMIN_ROLE) {
         UtilLib.checkNonZeroAddress(_vaultProxyImpl);
         vaultProxyImplementation = _vaultProxyImpl;
-        emit UpdatedVaultProxyImplementation(vaultProxyImplementation);
+        emit UpdatedVaultProxyImplementation(_vaultProxyImpl);
     }
 }
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L270-L271

### Emit `_nextQueuedValidatorIndex` instead of `nextQueuedValidatorIndex`

```solidity
File: /contracts/PermissionlessNodeRegistry.sol
270:        nextQueuedValidatorIndex = _nextQueuedValidatorIndex;
271:        emit UpdatedNextQueuedValidatorIndex(nextQueuedValidatorIndex);
```

```diff
     function updateNextQueuedValidatorIndex(uint256 _nextQueuedValidatorIndex) external {
         UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_POOL());
         nextQueuedValidatorIndex = _nextQueuedValidatorIndex;
-        emit UpdatedNextQueuedValidatorIndex(nextQueuedValidatorIndex);
+        emit UpdatedNextQueuedValidatorIndex(_nextQueuedValidatorIndex);
     }

```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L325-L329

### Emit `_inputKeyCountLimit` instead of `inputKeyCountLimit`

```solidity
File: /contracts/PermissionlessNodeRegistry.sol
327:        inputKeyCountLimit = _inputKeyCountLimit;
328:        emit UpdatedInputKeyCountLimit(inputKeyCountLimit);
```

```diff
-        emit UpdatedInputKeyCountLimit(inputKeyCountLimit);
+        emit UpdatedInputKeyCountLimit(_inputKeyCountLimit);
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L336-L340

### Emit `_maxNonTerminalKeyPerOperator` instead of `maxNonTerminalKeyPerOperator`

```solidity
File: /contracts/PermissionlessNodeRegistry.sol
338:        maxNonTerminalKeyPerOperator = _maxNonTerminalKeyPerOperator;
339:        emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);
```

```diff
-        emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);
+        emit UpdatedMaxNonTerminalKeyPerOperator(_maxNonTerminalKeyPerOperator)
```

## Take advantage of cached values to avoid reading from storage

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L20-L36

### Avoid reading state if we have an equivalent memory value

```solidity
File: /contracts/VaultProxy.sol
20:    function initialise(

34:        staderConfig = IStaderConfig(_staderConfig);
35:        owner = staderConfig.getAdmin();
36:    }
```

```diff
diff --git a/contracts/VaultProxy.sol b/contracts/VaultProxy.sol
index 115d018..a9d9a5c 100644
--- a/contracts/VaultProxy.sol
+++ b/contracts/VaultProxy.sol
@@ -32,7 +32,7 @@ contract VaultProxy is IVaultProxy {
         poolId = _poolId;
         id = _id;
         staderConfig = IStaderConfig(_staderConfig);
-        owner = staderConfig.getAdmin();
+        owner = IStaderConfig(_staderConfig).getAdmin();
     }
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L80-L91

### If a global/local variable is equal to a storage value, we can use the global/local one

```solidity
File: /contracts/Auction.sol
80:    function claimSD(uint256 lotId) external override {

83:        if (msg.sender != lotItem.highestBidder) revert notQualified();

87:        if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {
88:            revert SDTransferFailed();
89:        }
90:        emit SDClaimed(lotId, lotItem.highestBidder, lotItem.sdAmount);
```
In the above function, note the check on Line 83. We check that the `msg.sender `  is equal to `lotItem.highestBidder` . That means, the only way we get to the `transfer` call and `emit` block is if `msg.sender == lotItem.highestBidder` therefore we can refactor the code as follows to emit `msg.sender` which would cost ~3 gas instead of the storage `lotItem.highestBidder` which costs ~100 gas

```diff
diff --git a/contracts/Auction.sol b/contracts/Auction.sol
index ae5fd02..4b6bfe8 100644
--- a/contracts/Auction.sol
+++ b/contracts/Auction.sol
@@ -84,10 +84,10 @@ contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableU
         if (lotItem.sdClaimed) revert AlreadyClaimed();

         lotItem.sdClaimed = true;
-        if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {
+        if (!IERC20(staderConfig.getStaderToken()).transfer(msg.sender, lotItem.sdAmount)) {
             revert SDTransferFailed();
         }
-        emit SDClaimed(lotId, lotItem.highestBidder, lotItem.sdAmount);
+        emit SDClaimed(lotId, msg.sender, lotItem.sdAmount);
     }
```

## Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L691-L697
```solidity
File: /contracts/PermissionlessNodeRegistry.sol
691:    function isNonTerminalValidator(uint256 _validatorId) internal view returns (bool) {
692:        Validator memory validator = validatorRegistry[_validatorId];
693:        return
694:            !(validator.status == ValidatorStatus.WITHDRAWN ||
695:                validator.status == ValidatorStatus.FRONT_RUN ||
696:                validator.status == ValidatorStatus.INVALID_SIGNATURE);
697:    }
```


```diff
diff --git a/contracts/PermissionlessNodeRegistry.sol b/contracts/PermissionlessNodeRegistry.sol
index 9640556..b01b614 100644
--- a/contracts/PermissionlessNodeRegistry.sol
+++ b/contracts/PermissionlessNodeRegistry.sol
@@ -689,7 +689,7 @@ contract PermissionlessNodeRegistry is

     // checks if validator status enum is not withdrawn ,front run and invalid
signature
     function isNonTerminalValidator(uint256 _validatorId) internal view returns (bool) {
-        Validator memory validator = validatorRegistry[_validatorId];
+        Validator storage validator = validatorRegistry[_validatorId];
         return
             !(validator.status == ValidatorStatus.WITHDRAWN ||
                 validator.status == ValidatorStatus.FRONT_RUN ||
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L701-L704
```solidity
File: /contracts/PermissionlessNodeRegistry.sol
701:    function isActiveValidator(uint256 _validatorId) internal view returns (bool) {
702:        Validator memory validator = validatorRegistry[_validatorId];
703:        return validator.status == ValidatorStatus.DEPOSITED;
704:    }
```

```diff
diff --git a/contracts/PermissionlessNodeRegistry.sol b/contracts/PermissionlessNodeRegistry.sol
index 9640556..901fc5f 100644
--- a/contracts/PermissionlessNodeRegistry.sol
+++ b/contracts/PermissionlessNodeRegistry.sol
@@ -699,7 +699,7 @@ contract PermissionlessNodeRegistry is
     // checks if validator is active,
     //active validator are those having user deposit staked on beacon chain
     function isActiveValidator(uint256 _validatorId) internal view returns (bool) {
-        Validator memory validator = validatorRegistry[_validatorId];
+        Validator storage validator = validatorRegistry[_validatorId];
         return validator.status == ValidatorStatus.DEPOSITED;
     }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/UserWithdrawalManager.sol#L169

### We can use storage and then cache the values being used repeatedly

```solidity
File: /contracts/UserWithdrawalManager.sol
169:        UserWithdrawInfo memory userRequest = userWithdrawRequests[_requestId];
```

```diff
diff --git a/contracts/UserWithdrawalManager.sol b/contracts/UserWithdrawalManager.sol
index f563434..a1945f6 100644
--- a/contracts/UserWithdrawalManager.sol
+++ b/contracts/UserWithdrawalManager.sol
@@ -166,8 +166,9 @@ contract UserWithdrawalManager is
         if (_requestId >= nextRequestIdToFinalize) {
             revert requestIdNotFinalized(_requestId);
         }
-        UserWithdrawInfo memory userRequest = userWithdrawRequests[_requestId];
-        if (msg.sender != userRequest.owner) {
+        UserWithdrawInfo storage userRequest = userWithdrawRequests[_requestId]
;
+        address _owner = userRequest.owner;
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

## Leverage mapping and dot notation for struct assignment

We have a few methods we can use when assigning values to struct

- Type({field1: value1, field2: value2});
- Type(value1,value2);
- dot notation(structObject.field = value1);

```solidity
struct Example{
    uint a;
    uint b;
  }
  Example public example;

  function sample1(uint value1, uint value2) external {
    example.a = value1;
    example.b = value2;
  }
  function sample2(uint value1,uint value2) external{
      example = Example(value1, value2);
    }

  function sample3(uint value1, uint value2) external{
    example = Example({
      a : value1,
      b : value2
    });
  }

```

When we use the first two examples(sample1,sample2), the values are directly written to storage variable(example), however using the last method(sample3)
the compiler will allocate some memory to store the struct instance first before writing it to storage
Example({a:value1,b:value2}) will create new Example in memory to store the values a and b. Then once we have this in memory we need to write it to the storage var example.
That said, we can see where the gas difference would come from, the first two writes the values directly to storage while the last one needs to copy to memory first.


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L119-L139
```solidity
File: /contracts/SDCollateral.sol
131:        poolThresholdbyPoolId[_poolId] = PoolThresholdInfo({
132:            minThreshold: _minThreshold,
133:            maxThreshold: _maxThreshold,
134:            withdrawThreshold: _withdrawThreshold,
135:            units: _units
136:        });
```

```diff
diff --git a/contracts/SDCollateral.sol b/contracts/SDCollateral.sol
index 0b5e22b..4eb169c 100644
--- a/contracts/SDCollateral.sol
+++ b/contracts/SDCollateral.sol
@@ -127,13 +127,11 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
         if ((_minThreshold > _withdrawThreshold) || (_minThreshold > _maxThreshold)) {
             revert InvalidPoolLimit();
         }
-
-        poolThresholdbyPoolId[_poolId] = PoolThresholdInfo({
-            minThreshold: _minThreshold,
-            maxThreshold: _maxThreshold,
-            withdrawThreshold: _withdrawThreshold,
-            units: _units
-        });
+        PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];
+        poolThreshold.minThreshold = _minThreshold;
+        poolThreshold.maxThreshold = _maxThreshold;
+        poolThreshold.withdrawThreshold = _withdrawThreshold;
+        poolThreshold.units =  _units;

         emit UpdatedPoolThreshold(_poolId, _minThreshold, _withdrawThreshold);
     }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L661-L662

```solidity
File: /contracts/PermissionedNodeRegistry.sol
661:    function onboardOperator(string calldata _operatorName, address payable _operatorRewardAddress) internal {
662:        operatorStructById[nextOperatorId] = Operator(true, true, _operatorName, _operatorRewardAddress, msg.sender);
```

```diff
diff --git a/contracts/PermissionedNodeRegistry.sol b/contracts/PermissionedNodeRegistry.sol
index ca8aa81..ac45642 100644
--- a/contracts/PermissionedNodeRegistry.sol
+++ b/contracts/PermissionedNodeRegistry.sol
@@ -659,7 +659,13 @@ contract PermissionedNodeRegistry is
     }

     function onboardOperator(string calldata _operatorName, address payable _operatorRewardAddress) internal {
-        operatorStructById[nextOperatorId] = Operator(true, true, _operatorName, _operatorRewardAddress, msg.sender);
+        Operator storage operator = operatorStructById[nextOperatorId];
+        operator.active = true;
+        operator.optedForSocializingPool = true;
+        operator.operatorName = _operatorName;
+        operator.operatorRewardAddress = _operatorRewardAddress;
+        operator.operatorAddress = msg.sender;
+
         operatorIDByAddress[msg.sender] = nextOperatorId;
         socializingPoolStateChangeBlock[nextOperatorId] = block.number;
         nextOperatorId++;
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L598-L615
```solidity
File: /contracts/PermissionlessNodeRegistry.sol
603:        operatorStructById[nextOperatorId] = Operator(
604:            true,
605:            _optInForSocializingPool,
606:            _operatorName,
607:            _operatorRewardAddress,
608:            msg.sender
609:        );
```

```diff
diff --git a/contracts/PermissionlessNodeRegistry.sol b/contracts/PermissionlessNodeRegistry.sol
index 9640556..f7bb09a 100644
--- a/contracts/PermissionlessNodeRegistry.sol
+++ b/contracts/PermissionlessNodeRegistry.sol
@@ -600,13 +600,13 @@ contract PermissionlessNodeRegistry is
         string calldata _operatorName,
         address payable _operatorRewardAddress
     ) internal {
-        operatorStructById[nextOperatorId] = Operator(
-            true,
-            _optInForSocializingPool,
-            _operatorName,
-            _operatorRewardAddress,
-            msg.sender
-        );
+        Operator storage operatorbyStruct = operatorStructById[nextOperatorId];
+        operatorbyStruct.active = true;
+        operatorbyStruct.optedForSocializingPool = _optInForSocializingPool;
+        operatorbyStruct.operatorName = _operatorName;
+        operatorbyStruct.operatorRewardAddress = _operatorRewardAddress;
+        operatorbyStruct.operatorAddress = msg.sender;
+
         operatorIDByAddress[msg.sender] = nextOperatorId;
         socializingPoolStateChangeBlock[nextOperatorId] = block.number;
         nextOperatorId++;
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L48-L60

## Caching to local storage can be used for assignment not just reading values.

```solidity
File: /contracts/Auction.sol
48:    function createLot(uint256 _sdAmount) external override whenNotPaused {
49:        lots[nextLot].startBlock = block.number;
50:        lots[nextLot].endBlock = block.number + duration;
51:        lots[nextLot].sdAmount = _sdAmount;

53:        LotItem storage lotItem = lots[nextLot];

55:        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
56:            revert SDTransferFailed();
57:        }
58:        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
59:        nextLot++;
60:    }
```

In the above function, we are writing values to `lots[nextLot]` then cache it to a local storage `LotItem storage lotItem = lots[nextLot];` 
We then use this local storage variable to read individual values. However, it would be more efficient to actually use the local storage from the word go ie use it when writing values and when reading values.
See refactored code below: note I've added some more optimizations on the emit part.

```diff
diff --git a/contracts/Auction.sol b/contracts/Auction.sol
index ae5fd02..b1cb62b 100644
--- a/contracts/Auction.sol
+++ b/contracts/Auction.sol
@@ -46,16 +46,17 @@ contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableU
     }

     function createLot(uint256 _sdAmount) external override whenNotPaused {
-        lots[nextLot].startBlock = block.number;
-        lots[nextLot].endBlock = block.number + duration;
-        lots[nextLot].sdAmount = _sdAmount;
-
         LotItem storage lotItem = lots[nextLot];
+        lotItem.startBlock = block.number;
+        lotItem.endBlock = block.number + duration;
+        lotItem.sdAmount = _sdAmount;
+
+

         if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
             revert SDTransferFailed();
         }
-        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
+        emit LotCreated(nextLot, _sdAmount, block.number, lotItem.endBlock, bidIncrement);
         nextLot++;
     }

```

## Use calldata instead of memory for function parameters

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L119-L125
```solidity
File: /contracts/SDCollateral.sol
119:    function updatePoolThreshold(
120:        uint8 _poolId,
121:        uint256 _minThreshold,
122:        uint256 _maxThreshold,
123:        uint256 _withdrawThreshold,
124:        string memory _units
125:    ) external override {
```


```diff
diff --git a/contracts/SDCollateral.sol b/contracts/SDCollateral.sol
index 0b5e22b..5cab33b 100644
--- a/contracts/SDCollateral.sol
+++ b/contracts/SDCollateral.sol
@@ -121,7 +121,7 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
         uint256 _minThreshold,
         uint256 _maxThreshold,
         uint256 _withdrawThreshold,
-        string memory _units
+        string calldata _units
     ) external override {
         UtilLib.onlyManagerRole(msg.sender, staderConfig);
         if ((_minThreshold > _withdrawThreshold) || (_minThreshold > _maxThreshold)) {
```


## Avoiding Unnecessary SLOAD in Solidity Subtraction Operations

### Use the cached value: `operatorSDBalance[operator]` has been cached already (save 1 SLOAD ~100 gas)

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L58-L65
```solidity
File: /contracts/SDCollateral.sol
58:    function withdraw(uint256 _requestedSD) external override {
59:        address operator = msg.sender;
60:        uint256 opSDBalance = operatorSDBalance[operator];

62:        if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
63:            revert InsufficientSDToWithdraw(opSDBalance);
64:        }
65:        operatorSDBalance[operator] -= _requestedSD;
```

On Line 60, we cache the value of `operatorSDBalance[operator]`. When doing the subtraction on line 65, we can rewrite the operation as follows to avoid reading from storage twice when we already have a memory value.

```diff
diff --git a/contracts/SDCollateral.sol b/contracts/SDCollateral.sol
index 0b5e22b..d978e13 100644
--- a/contracts/SDCollateral.sol
+++ b/contracts/SDCollateral.sol
@@ -62,7 +62,7 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
         if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
             revert InsufficientSDToWithdraw(opSDBalance);
         }
-        operatorSDBalance[operator] -= _requestedSD;
+        operatorSDBalance[operator] = opSDBalance -  _requestedSD;

```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L90-L96

### `operatorSDBalance[_operator]` has been cached into `sdBalance` and the cached value should be used

```solidity
File: /contracts/SDCollateral.sol
90:    function slashSD(address _operator, uint256 _sdToSlash) internal {
91:        uint256 sdBalance = operatorSDBalance[_operator];
92:        uint256 sdSlashed = Math.min(_sdToSlash, sdBalance);
93:        if (sdSlashed == 0) {
94:            return;
95:        }
96:        operatorSDBalance[_operator] -= sdSlashed;
```
When doing the subtraction,  using the short form `-=` would be interpreted as `operatorSDBalance[_operator] = operatorSDBalance[_operator] - sdSlashed;` meaning we would be making an SLOAD for the variable `operatorSDBalance[_operator] ` which is a waste of gas as we have a memory value equal to that variable

```diff
diff --git a/contracts/SDCollateral.sol b/contracts/SDCollateral.sol
index 0b5e22b..6cceecb 100644
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

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L120-L135

## We can save 1 SLOAD plus a few gas cost for the subtractions

```solidity
File: /contracts/Auction.sol
120:    function withdrawUnselectedBid(uint256 lotId) external override nonReentrant {

125:		uint256 withdrawalAmount = lotItem.bids[msg.sender];
126:       if (withdrawalAmount == 0) revert InSufficientETH();

128:        lotItem.bids[msg.sender] -= withdrawalAmount;
```
We read `lotItem.bids[msg.sender]` and store it in `withdrawalAmount`, we then do the subtraction `lotItem.bids[msg.sender] -= withdrawalAmount` which just would simply be equal to setting our state variable `lotItem.bids[msg.sender]` to zero.

```diff
diff --git a/contracts/Auction.sol b/contracts/Auction.sol
index ae5fd02..663cbac 100644
--- a/contracts/Auction.sol
+++ b/contracts/Auction.sol
@@ -125,7 +125,7 @@ contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableU
         uint256 withdrawalAmount = lotItem.bids[msg.sender];
         if (withdrawalAmount == 0) revert InSufficientETH();

-        lotItem.bids[msg.sender] -= withdrawalAmount;
+        delete lotItem.bids[msg.sender];

         // send the funds
         (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/OperatorRewardsCollector.sol#L46-L54

## We can save SLOAD here and some small operation costs(~180 gas)

```solidity
File: /contracts/OperatorRewardsCollector.sol
46:    function claim() external whenNotPaused {
47:        address operator = msg.sender;
48:        uint256 amount = balances[operator];
49:        balances[operator] -= amount;
```

In the above function, we are reading the value of `balances[operator]` and saving it to a local variable `amount`. We then write to the storage variable `balances[operator]` the difference between the value of `balances[operator]` and `amount`. However , since we cached the value of the state variable in `amount` the subtraction would always result in 0.
As the subtraction forces us to read from state again, it would be cheaper to just delete the variable/set it to zero as shown below.

```diff
diff --git a/contracts/OperatorRewardsCollector.sol b/contracts/OperatorRewardsCollector.sol
index e80db8a..4d22d4c 100644
--- a/contracts/OperatorRewardsCollector.sol
+++ b/contracts/OperatorRewardsCollector.sol
@@ -46,7 +46,7 @@ contract OperatorRewardsCollector is
     function claim() external whenNotPaused {
         address operator = msg.sender;
         uint256 amount = balances[operator];
-        balances[operator] -= amount;
+        delete balances[operator];

         address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig);
         UtilLib.sendValue(operatorRewardsAddr, amount);
```



## Optimizing check order for cost efficient function execution

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### Validate all values before reading storage

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L93-L104
```solidity
File: /contracts/Auction.sol
93:    function transferHighestBidToSSPM(uint256 lotId) external override nonReentrant {
94:        LotItem storage lotItem = lots[lotId];
95:        uint256 ethAmount = lotItem.highestBidAmount;

97:        if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
98:        if (ethAmount == 0) revert NoBidPlaced();
```
In the above function, we read `lotItem.highestBidAmount`  and cache it in the local variable `ethAmount` . We then validate `block.number <= lotItem.endBlock` and revert if our requirements are not met. If we do revert, the gas spent reading the first variable  `lotItem.highestBidAmount`  would go to waste. We can refactor the code as follows to avoid wasting gas on state reads incase of a revert

```diff
diff --git a/contracts/Auction.sol b/contracts/Auction.sol
index ae5fd02..120e8fb 100644
--- a/contracts/Auction.sol
+++ b/contracts/Auction.sol
@@ -92,11 +92,11 @@ contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableU

     function transferHighestBidToSSPM(uint256 lotId) external override nonReentrant {
         LotItem storage lotItem = lots[lotId];
-        uint256 ethAmount = lotItem.highestBidAmount;
-
         if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
-        if (ethAmount == 0) revert NoBidPlaced();
         if (lotItem.ethExtracted) revert AlreadyClaimed();
+
+        uint256 ethAmount = lotItem.highestBidAmount;
+        if (ethAmount == 0) revert NoBidPlaced();

         lotItem.ethExtracted = true;
         IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveEthFromAuction{value: ethAmount}();
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L144-L149

### Reorder the checks to have the one that doesn't read from storage come first

```solidity
File: /contracts/Auction.sol
146:    function updateDuration(uint256 _duration) external override {
147:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
148:        if (_duration < MIN_AUCTION_DURATION) revert ShortDuration();
```
As it is, if we end up reverting on the second check, we would waste the gas spent doing an SLOAD ~2100 gas since the first check reads from state. As the second one doesn't read any state, the gas consumed would be way less making it more efficient to check it first

```diff
     function updateDuration(uint256 _duration) external override {
-        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         if (_duration < MIN_AUCTION_DURATION) revert ShortDuration();
+        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         duration = _duration;
         emit AuctionDurationUpdated(duration);
     }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L236-L245

### Reorder the checks to have the one not reading from storage come first

```solidity
File: /contracts/PermissionedPool.sol
236:    function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {
237:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
238:        if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {
239:            revert InvalidCommission();
240:        }
```
The first check would read from storage(`staderConfig`) which is too expensive. If we end up reverting on the second check, the gas used in reading the state variable would be a waste. We can refactor the code to minimize the gas consumed in case of a revert on the check `_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS`

```diff
     function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {
-        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {
             revert InvalidCommission();
         }
+        UtilLib.onlyManagerRole(msg.sender, staderConfig);
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L89-L99

### Should we re implement the lib or moving is enough

```solidity
File: /contracts/PermissionlessNodeRegistry.sol
89:    function onboardNodeOperator(
90:        bool _optInForSocializingPool,
91:        string calldata _operatorName,
92:        address payable _operatorRewardAddress
93:    ) external override whenNotPaused returns (address feeRecipientAddress) {
94:        address poolUtils = staderConfig.getPoolUtils();
95:        if (IPoolUtils(poolUtils).poolAddressById(POOL_ID) != staderConfig.getPermissionlessPool()) {
96:            revert DuplicatePoolIDOrPoolNotAdded();
97:        }
98:        IPoolUtils(poolUtils).onlyValidName(_operatorName);
99:        UtilLib.checkNonZeroAddress(_operatorRewardAddress);
```


```diff
--- a/contracts/PermissionlessNodeRegistry.sol
+++ b/contracts/PermissionlessNodeRegistry.sol
@@ -91,12 +91,13 @@ contract PermissionlessNodeRegistry is
         string calldata _operatorName,
         address payable _operatorRewardAddress
     ) external override whenNotPaused returns (address feeRecipientAddress) {
+
+        UtilLib.checkNonZeroAddress(_operatorRewardAddress);
         address poolUtils = staderConfig.getPoolUtils();
         if (IPoolUtils(poolUtils).poolAddressById(POOL_ID) != staderConfig.getPermissionlessPool()) {
             revert DuplicatePoolIDOrPoolNotAdded();
         }
         IPoolUtils(poolUtils).onlyValidName(_operatorName);
-        UtilLib.checkNonZeroAddress(_operatorRewardAddress);

```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L366-L368

### External calls are expensive compared to the nonzero checks

```solidity
File: /contracts/PermissionlessNodeRegistry.sol
366:    function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {
367:        IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);
368:        UtilLib.checkNonZeroAddress(_rewardAddress);
```

```diff
     function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {
-        IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);
         UtilLib.checkNonZeroAddress(_rewardAddress);
+        IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessPool.sol#L66-L75

### As the first check reads from storage, it should be done after cheaper checks

```solidity
File: /contracts/PermissionlessPool.sol
66:    function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {
67:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
68:        if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {
69:            revert InvalidCommission();
70:        }
```
Consider moving the second check above, to minimize gas consumed in case of a revert on `_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS`

```diff
diff --git a/contracts/PermissionlessPool.sol b/contracts/PermissionlessPool.sol
index 046761d..300ebac 100644
--- a/contracts/PermissionlessPool.sol
+++ b/contracts/PermissionlessPool.sol
@@ -64,10 +64,10 @@ contract PermissionlessPool is IStaderPoolBase, Initializable, AccessControlUpgr

     /// @inheritdoc IStaderPoolBase
     function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {
-        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {
             revert InvalidCommission();
         }
+        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         protocolFee = _protocolFee;
         operatorFee = _operatorFee;
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolUtils.sol#L40-L42

### Cheaper checks should be done first

```solidity
File: /contracts/PoolUtils.sol
40:    function addNewPool(uint8 _poolId, address _poolAddress) external override {
41:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
42:        UtilLib.checkNonZeroAddress(_poolAddress);
```
The nonzero check does not read from storage while the onlyManager check reads state variable. Reorder the checks to minimize gas consumed if we do revert on the nonzero check
```diff
     function addNewPool(uint8 _poolId, address _poolAddress) external override {
-        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         UtilLib.checkNonZeroAddress(_poolAddress);
+        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         verifyNewPool(_poolId, _poolAddress);
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L119-L129

### Don't read any state if we might revert on function parameters and constants

```solidity
File: /contracts/SDCollateral.sol
119:    function updatePoolThreshold(
120:        uint8 _poolId,
121:        uint256 _minThreshold,
122:        uint256 _maxThreshold,
123:        uint256 _withdrawThreshold,
124:        string memory _units
125:    ) external override {
126:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
127:        if ((_minThreshold > _withdrawThreshold) || (_minThreshold > _maxThreshold)) {
128:            revert InvalidPoolLimit();
129:        }
```
As the second check only reads memory variables and constants, it is cheaper compared to the first check that reads state variable. Reorder the checks to have the cheaper check first

```diff
diff --git a/contracts/SDCollateral.sol b/contracts/SDCollateral.sol
index 0b5e22b..991cc85 100644
--- a/contracts/SDCollateral.sol
+++ b/contracts/SDCollateral.sol
@@ -123,10 +123,10 @@ contract SDCollateral is ISDCollateral, Initializable, AccessControlUpgradeable,
         uint256 _withdrawThreshold,
         string memory _units
     ) external override {
-        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         if ((_minThreshold > _withdrawThreshold) || (_minThreshold > _maxThreshold)) {
             revert InvalidPoolLimit();
         }
+        UtilLib.onlyManagerRole(msg.sender, staderConfig);
```



https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderInsuranceFund.sol#L60-L64
```solidity
File: /contracts/StaderInsuranceFund.sol
60:    function reimburseUserFund(uint256 _amount) external override nonReentrant {
61:        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());
62:        if (address(this).balance < _amount) {
63:            revert InSufficientBalance();
64:        }
```

```diff
      */
     function reimburseUserFund(uint256 _amount) external override nonReentrant {
-        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());
         if (address(this).balance < _amount) {
             revert InSufficientBalance();
         }
+        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());
         IPermissionedPool(staderConfig.getPermissionedPool()).receiveInsuranceFund{value: _amount}();
     }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L81-L83

### The second check does not read from state thus should be done first

```solidity
File: /contracts/StaderOracle.sol
81:    function addTrustedNode(address _nodeAddress) external override {
82:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
83:        UtilLib.checkNonZeroAddress(_nodeAddress);
```

```diff
     function addTrustedNode(address _nodeAddress) external override {
-        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         UtilLib.checkNonZeroAddress(_nodeAddress);
+        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         if (isTrustedNode[_nodeAddress]) {
             revert NodeAlreadyTrusted();
         }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L94-L96

### Similar to the above

```solidity
File: /contracts/StaderOracle.sol
94:    function removeTrustedNode(address _nodeAddress) external override {
95:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
96:        UtilLib.checkNonZeroAddress(_nodeAddress);
```

```diff
     function removeTrustedNode(address _nodeAddress) external override {
-        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         UtilLib.checkNonZeroAddress(_nodeAddress);
+        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         if (!isTrustedNode[_nodeAddress]) {
             revert NodeNotTrusted();
         }
```



https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L515-L519

### The first checks reads state while the second doesn't , reorder them

```solidity
File: /contracts/StaderOracle.sol
515:    function setERUpdateFrequency(uint256 _updateFrequency) external override {
516:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
517:        if (_updateFrequency > MAX_ER_UPDATE_FREQUENCY) {
518:            revert InvalidUpdate();
519:        }
```

```diff

     function setERUpdateFrequency(uint256 _updateFrequency) external override {
-        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         if (_updateFrequency > MAX_ER_UPDATE_FREQUENCY) {
             revert InvalidUpdate();
         }
+        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         setUpdateFrequency(ETHX_ER_UF, _updateFrequency);
     }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L530-L534

### checks for function params should come first

```solidity
File: /contracts/StaderOracle.sol
530:    function updateERChangeLimit(uint256 _erChangeLimit) external override {
531:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
532:        if (_erChangeLimit == 0 || _erChangeLimit > ER_CHANGE_MAX_BPS) {
533:            revert ERPermissibleChangeOutofBounds();
534:        }
```

```diff
     function updateERChangeLimit(uint256 _erChangeLimit) external override {
-        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         if (_erChangeLimit == 0 || _erChangeLimit > ER_CHANGE_MAX_BPS) {
             revert ERPermissibleChangeOutofBounds();
         }
+        UtilLib.onlyManagerRole(msg.sender, staderConfig);
         erChangeLimit = _erChangeLimit;
         emit UpdatedERChangeLimit(erChangeLimit);
     }
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L26-L29

### The second check is cheaper and should be done first

```solidity
File: /contracts/VaultProxy.sol
26:        if (isInitialized) {
27:            revert AlreadyInitialized();
28:        }
29:        UtilLib.checkNonZeroAddress(_staderConfig);
```

```diff
diff --git a/contracts/VaultProxy.sol b/contracts/VaultProxy.sol
index 115d018..73afb2f 100644
--- a/contracts/VaultProxy.sol
+++ b/contracts/VaultProxy.sol
@@ -23,10 +23,10 @@ contract VaultProxy is IVaultProxy {
         uint256 _id,
         address _staderConfig
     ) external {
+        UtilLib.checkNonZeroAddress(_staderConfig);
         if (isInitialized) {
             revert AlreadyInitialized();
         }
-        UtilLib.checkNonZeroAddress(_staderConfig);
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ValidatorWithdrawalVault.sol#L34-L41

### Do sanity checks immediately, Can verify `totalRewards` before making external calls

```solidity
File: /contracts/ValidatorWithdrawalVault.sol
34:        uint256 totalRewards = address(this).balance;
35:        if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
36:            emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
37:            revert InvalidRewardAmount();
38:        }
39:        if (totalRewards == 0) {
40:            revert NotEnoughRewardToDistribute();
41:        }
```


```diff
diff --git a/contracts/ValidatorWithdrawalVault.sol b/contracts/ValidatorWithdrawalVault.sol
index eb3de50..ab1a229 100644
--- a/contracts/ValidatorWithdrawalVault.sol
+++ b/contracts/ValidatorWithdrawalVault.sol
@@ -28,17 +28,19 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
     }

     function distributeRewards() external override {
+        uint256 totalRewards = address(this).balance;
+        if (totalRewards == 0) {
+            revert NotEnoughRewardToDistribute();
+        }
         uint8 poolId = VaultProxy(payable(address(this))).poolId();
         uint256 validatorId = VaultProxy(payable(address(this))).id();
         IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
-        uint256 totalRewards = address(this).balance;
+
         if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
             emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
             revert InvalidRewardAmount();
         }
-        if (totalRewards == 0) {
-            revert NotEnoughRewardToDistribute();
-        }
+
         (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
             .calculateRewardShare(poolId, totalRewards);

```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/NodeELRewardVault.sol#L24-L31

### Validate all values before doing any external calls

```solidity
File: /contracts/NodeELRewardVault.sol
24:    function withdraw() external override {
25:        uint8 poolId = IVaultProxy(address(this)).poolId();
26:        uint256 operatorId = IVaultProxy(address(this)).id();
27:        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
28:        uint256 totalRewards = address(this).balance;
29:        if (totalRewards == 0) {
30:            revert NotEnoughRewardToWithdraw();
31:        }
```
The above function makes a few external calls to set some variables. We then read the contract's balance and revert in case the balance ie `totalRewards` is 0. Instead of doing all the previous external calls and maybe revert on the check of `totalRewards == 0` we should first validate the `totalRewards` and revert immediately if condition not satisfied.

```diff
diff --git a/contracts/NodeELRewardVault.sol b/contracts/NodeELRewardVault.sol
index b10417a..81c794f 100644
--- a/contracts/NodeELRewardVault.sol
+++ b/contracts/NodeELRewardVault.sol
@@ -22,13 +22,13 @@ contract NodeELRewardVault is INodeELRewardVault {
     }

     function withdraw() external override {
-        uint8 poolId = IVaultProxy(address(this)).poolId();
-        uint256 operatorId = IVaultProxy(address(this)).id();
-        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
         uint256 totalRewards = address(this).balance;
         if (totalRewards == 0) {
             revert NotEnoughRewardToWithdraw();
         }
+        uint8 poolId = IVaultProxy(address(this)).poolId();
+        uint256 operatorId = IVaultProxy(address(this)).id();
+        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
         (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
             .calculateRewardShare(poolId, totalRewards);

```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L112-L117

### Check functional params first being doing any external calls

```solidity
File: /contracts/PermissionedNodeRegistry.sol
112:        address poolUtils = staderConfig.getPoolUtils();
113:        if (IPoolUtils(poolUtils).poolAddressById(POOL_ID) != staderConfig.getPermissionedPool()) {
114:            revert DuplicatePoolIDOrPoolNotAdded();
115:        }
116:        IPoolUtils(poolUtils).onlyValidName(_operatorName);
117:        UtilLib.checkNonZeroAddress(_operatorRewardAddress);
```


```diff
diff --git a/contracts/PermissionedNodeRegistry.sol b/contracts/PermissionedNodeRegistry.sol
index ca8aa81..f91932c 100644
--- a/contracts/PermissionedNodeRegistry.sol
+++ b/contracts/PermissionedNodeRegistry.sol
@@ -109,12 +109,12 @@ contract PermissionedNodeRegistry is
         whenNotPaused
         returns (address feeRecipientAddress)
     {
+        UtilLib.checkNonZeroAddress(_operatorRewardAddress);
         address poolUtils = staderConfig.getPoolUtils();
         if (IPoolUtils(poolUtils).poolAddressById(POOL_ID) != staderConfig.getPermissionedPool()) {
             revert DuplicatePoolIDOrPoolNotAdded();
         }
         IPoolUtils(poolUtils).onlyValidName(_operatorName);
-        UtilLib.checkNonZeroAddress(_operatorRewardAddress);
         if (nextOperatorId > maxOperatorId) {
             revert MaxOperatorLimitReached();
         }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L401-L404

### Move the NonZero check to the top

```solidity
File: /contracts/PermissionedNodeRegistry.sol
401:    function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {
402:        IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);
403:        UtilLib.checkNonZeroAddress(_rewardAddress);
```

```diff
     function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {
-        IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);
         UtilLib.checkNonZeroAddress(_rewardAddress);
+        IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName);
```


## The modifier can be rewritten to save some gas

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L75-L80
```solidity
File: /contracts/VaultProxy.sol
75:    modifier onlyOwner() {
76:        if (msg.sender != owner) {
77:            revert CallerNotOwner();
78:        }
79:        _;
80:    }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L57-L58
```solidity
File: /contracts/VaultProxy.sol
57:    function updateStaderConfig(address _staderConfig) external override onlyOwner {
58:        UtilLib.checkNonZeroAddress(_staderConfig);
```
Note we have a check for `nonZeroAddress`, as this is a library call, we'd just spend around 20 gas for the JUMPS, however note the modifier has a check that reads from state ~100 gas. If we end up reverting on the `nonZeroAddress`  check , it means the gas spent on the modifier check would be wasted. To avoid wasting too much gas, we can inline the modifier and have the modifier check come after the `nonZeroAddress` check

```diff
diff --git a/contracts/VaultProxy.sol b/contracts/VaultProxy.sol
index 115d018..6da246c 100644
--- a/contracts/VaultProxy.sol
+++ b/contracts/VaultProxy.sol
@@ -54,8 +54,11 @@ contract VaultProxy is IVaultProxy {
      * @dev only owner can call
      * @param _staderConfig address of updated staderConfig
      */
-    function updateStaderConfig(address _staderConfig) external override onlyOwner {
+    function updateStaderConfig(address _staderConfig) external override {
         UtilLib.checkNonZeroAddress(_staderConfig);
+        if (msg.sender != owner) {
+            revert CallerNotOwner();
+        }
         staderConfig = IStaderConfig(_staderConfig);
         emit UpdatedStaderConfig(_staderConfig);
     }
```

**Similar instance to the above**
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L68-L72
```solidity
File: /contracts/VaultProxy.sol
68:    function updateOwner(address _owner) external override onlyOwner {
69:        UtilLib.checkNonZeroAddress(_owner);
```


## Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L190
```solidity
File: /contracts/SDCollateral.sol
190:        return (sdBalance >= minSDToBond ? 0 : minSDToBond - sdBalance);
```
The operation `minSDToBond - sdBalance` cannot underflow as it would be evaluated if `sdBalance` is less than `minSDToBond`

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ValidatorWithdrawalVault.sol#L72
```solidity
File: /contracts/ValidatorWithdrawalVault.sol
72:        operatorShare = operatorShare - penaltyAmount;
```
The operation `operatorShare - penaltyAmount` cannot underflow due to the check on [Line 66](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ValidatorWithdrawalVault.sol#L66) that sets `penaltyAmount` equal to `operatorShare` if `operatorShare` is less than `penaltyAmount`

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ValidatorWithdrawalVault.sol#L108
```solidity
File: /contracts/ValidatorWithdrawalVault.sol
108:            _operatorShare = contractBalance - _userShare;
```
The operation `contractBalance - _userShare` cannot underflow as it would only be performed if `contractBalance` is not less than `usersETH`. Note that `_userShare` is equal to `usersETH`. If `contractBalance` < `usersETH` we would fall in the first block of the if statement on [Line 103](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ValidatorWithdrawalVault.sol#LL103C32-L103C40)

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ValidatorWithdrawalVault.sol#L111
```solidity
File: /contracts/ValidatorWithdrawalVault.sol
111:            totalRewards = contractBalance - TOTAL_STAKED_ETH;
```
The operation `contractBalance - TOTAL_STAKED_ETH` cannot underflow as it would only be performed if `contractBalance` is greater than `TOTAL_STAKED_ETH` due to the check on [Line 106](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ValidatorWithdrawalVault.sol#L106)

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L224
```solidity
File: /contracts/PermissionedNodeRegistry.sol
224:            uint256 remainingValidatorsToDeposit = _numValidators - totalValidatorToDeposit;
```
The operation `_numValidators - totalValidatorToDeposit` cannot underflow due to the check on [Line 222](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L222) that ensures that `_numValidators` is greater than `totalValidatorToDeposit` before doing the subtraction 


## Don't cache function params (save ~5 gas)

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L76-L82
```solidity
File: /contracts/PoolSelector.sol
76:    function poolAllocationForExcessETHDeposit(uint256 _excessETHAmount)

82:        uint256 ethToDeposit = _excessETHAmount;
```

It is already cheap using the parameter directly, caching only increases gas cost

```diff
diff --git a/contracts/PoolSelector.sol b/contracts/PoolSelector.sol
index bdb1db9..34574f6 100644
--- a/contracts/PoolSelector.sol
+++ b/contracts/PoolSelector.sol
@@ -90,17 +90,17 @@ contract PoolSelector is IPoolSelector, Initializable, AccessControlUpgradeable
         for (uint256 j; j < poolCount; ) {
             uint256 poolCapacity = poolUtils.getQueuedValidatorCountByPool(poolIdArray[i]);
             uint256 poolDepositSize = ETH_PER_NODE - poolUtils.getCollateralETH(poolIdArray[i]);
-            uint256 remainingValidatorsToDeposit = ethToDeposit / poolDepositSize;
+            uint256 remainingValidatorsToDeposit = _excessETHAmount / poolDepositSize;
             selectedPoolCapacity[i] = Math.min(
                 poolAllocationMaxSize - selectedValidatorCount,
                 Math.min(poolCapacity, remainingValidatorsToDeposit)
             );
             selectedValidatorCount += selectedPoolCapacity[i];
-            ethToDeposit -= selectedPoolCapacity[i] * poolDepositSize;
+            _excessETHAmount -= selectedPoolCapacity[i] * poolDepositSize;
             i = (i + 1) % poolCount;
             //For ethToDeposit < ETH_PER_NODE, we will be able to at best deposit one more validator
             //but that will introduce complex logic, hence we are not solving that
-            if (ethToDeposit < ETH_PER_NODE || selectedValidatorCount >= poolAllocationMaxSize) {
+            if (_excessETHAmount < ETH_PER_NODE || selectedValidatorCount >= poolAllocationMaxSize) {
                 poolIdArrayIndexForExcessDeposit = i;
                 break;
             }
```

## Short hand if

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ValidatorWithdrawalVault.sol#L35-L38
```solidity
File: /contracts/ValidatorWithdrawalVault.sol
35:        if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
36:            emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
37:            revert InvalidRewardAmount();
38:        }
```

```diff
diff --git a/contracts/ValidatorWithdrawalVault.sol b/contracts/ValidatorWithdrawalVault.sol
index eb3de50..87c6167 100644
--- a/contracts/ValidatorWithdrawalVault.sol
+++ b/contracts/ValidatorWithdrawalVault.sol
@@ -32,10 +32,12 @@ contract ValidatorWithdrawalVault is IValidatorWithdrawalVault {
         uint256 validatorId = VaultProxy(payable(address(this))).id();
         IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
         uint256 totalRewards = address(this).balance;
-        if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > stader
Config.getRewardsThreshold()) {
+        if (!staderConfig.onlyOperatorRole(msg.sender)){
+            if (totalRewards > staderConfig.getRewardsThreshold()) {
             emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
             revert InvalidRewardAmount();
         }
+        }
         if (totalRewards == 0) {
             revert NotEnoughRewardToDistribute();
         }
```

## Caching calldata increases gas instead of saving it(Saves ~5 gas)

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L119-L123
```solidity
File: /contracts/PoolSelector.sol
119:    function updatePoolWeights(uint256[] calldata _poolTargets) external {
120:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
121:        uint8[] memory poolIdArray = IPoolUtils(staderConfig.getPoolUtils()).getPoolIdArray();
122:        uint256 poolCount = poolIdArray.length;
123:        uint256 poolTargetLength = _poolTargets.length;

125:        if (poolCount != poolTargetLength) {
126:            revert InvalidNewTargetInput();
127:        }

130:        for (uint256 i = 0; i < poolTargetLength; i++) {
```

```diff
diff --git a/contracts/PoolSelector.sol b/contracts/PoolSelector.sol
index bdb1db9..955e1ef 100644
--- a/contracts/PoolSelector.sol
+++ b/contracts/PoolSelector.sol
@@ -122,12 +122,12 @@ contract PoolSelector is IPoolSelector, Initializable, AccessControlUpgradeable
         uint256 poolCount = poolIdArray.length;
         uint256 poolTargetLength = _poolTargets.length;

-        if (poolCount != poolTargetLength) {
+        if (poolCount !=  _poolTargets.length) {
             revert InvalidNewTargetInput();
         }

         uint256 totalWeight;
-        for (uint256 i = 0; i < poolTargetLength; i++) {
+        for (uint256 i = 0; i <  _poolTargets.length; i++) {
             totalWeight += _poolTargets[i];
             poolWeights[poolIdArray[i]] = _poolTargets[i];
             emit UpdatedPoolWeight(poolIdArray[i], _poolTargets[i]);
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L88-L97
```solidity
File: /contracts/PermissionedNodeRegistry.sol
88:    function whitelistPermissionedNOs(address[] calldata _permissionedNOs) external override {
89:        UtilLib.onlyManagerRole(msg.sender, staderConfig);
90:        uint256 permissionedNosLength = _permissionedNOs.length;
91:        for (uint256 i = 0; i < permissionedNosLength; i++) {
```

```diff
     function whitelistPermissionedNOs(address[] calldata _permissionedNOs) exte
rnal override {
         UtilLib.onlyManagerRole(msg.sender, staderConfig);
-        uint256 permissionedNosLength = _permissionedNOs.length;
-        for (uint256 i = 0; i < permissionedNosLength; i++) {
+        for (uint256 i = 0; i < _permissionedNOs.length; i++) {
```

**Other Instances: No diffs, see audit tags**
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L183-L191
```solidity
File: /contracts/PermissionlessNodeRegistry.sol
//@audit: Do not cache the length of _readyToDepositPubkey,_frontRunPubkey,_invalidSignaturePubkey
183:    function markValidatorReadyToDeposit(
184:        bytes[] calldata _readyToDepositPubkey,
185:        bytes[] calldata _frontRunPubkey,
186:        bytes[] calldata _invalidSignaturePubkey
187:    ) external override nonReentrant whenNotPaused {
188:        UtilLib.onlyOperatorRole(msg.sender, staderConfig);
189:        uint256 readyToDepositValidatorsLength = _readyToDepositPubkey.length;
190:        uint256 frontRunValidatorsLength = _frontRunPubkey.length;
191:        uint256 invalidSignatureValidatorsLength = _invalidSignaturePubkey.length;
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L241-L243
```solidity
File: /contracts/PermissionlessNodeRegistry.sol
241:    function withdrawnValidators(bytes[] calldata _pubkeys) external override {
//@audit: Don't cache the length of _pubkeys
243:        uint256 withdrawnValidatorCount = _pubkeys.length;
```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#L144
```solidity
File: /contracts/SocializingPool.sol

//@audit: Don't cache the length of _index
144:        uint256 indexLength = _index.length;
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L311-L313
```solidity
File: /contracts/PermissionedNodeRegistry.sol
311:    function withdrawnValidators(bytes[] calldata _pubkeys) external override {
//@audit: Don't cache the length here
313:        uint256 withdrawnValidatorCount = _pubkeys.length;
```


https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L129-L137
```solidity
File: /contracts/PermissionedPool.sol
129:    function fullDepositOnBeaconChain(bytes[] calldata _pubkey) external nonReentrant {
//@audit: Don't cache the length of _pubkey
134:        uint256 pubkeyCount = _pubkey.length;

137:        for (uint256 i; i < pubkeyCount; ) {
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessPool.sol#L85-L94
```solidity
File: /contracts/PermissionlessPool.sol
85:    function preDepositOnBeaconChain(
86:        bytes[] calldata _pubkey,

90:    ) external payable nonReentrant {

//@audit: Don't cache the lenght of _pubkey
93:        uint256 pubkeyCount = _pubkey.length;
94:        for (uint256 i; i < pubkeyCount; ) {
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolUtils.sol#L72-L75
```solidity
File: /contracts/PoolUtils.sol
72:    function processValidatorExitList(bytes[] calldata _pubkeys) external override {
//@audit: Don't cache the length of _pubkeys
74:        uint256 exitValidatorCount = _pubkeys.length;
75:        for (uint256 i; i < exitValidatorCount; ) {
```

## Caching global variables/ `msg.sender` increases gas

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/OperatorRewardsCollector.sol#L46-L54
```solidity
File: /contracts/OperatorRewardsCollector.sol
46:    function claim() external whenNotPaused {
47:        address operator = msg.sender;
48:        uint256 amount = balances[operator];
49:        balances[operator] -= amount;

51:        address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig);
52:        UtilLib.sendValue(operatorRewardsAddr, amount);
53:        emit Claimed(operatorRewardsAddr, amount);
54:    }
```


```diff
diff --git a/contracts/OperatorRewardsCollector.sol b/contracts/OperatorRewardsCollector.sol
index e80db8a..db08fa8 100644
--- a/contracts/OperatorRewardsCollector.sol
+++ b/contracts/OperatorRewardsCollector.sol
@@ -44,9 +44,8 @@ contract OperatorRewardsCollector is
     }

     function claim() external whenNotPaused {
-        address operator = msg.sender;
-        uint256 amount = balances[operator];
-        balances[operator] -= amount;
+        uint256 amount = balances[msg.sender];
+        balances[msg.sender] -= amount;

         address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig);
         UtilLib.sendValue(operatorRewardsAddr, amount);
```


## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.



