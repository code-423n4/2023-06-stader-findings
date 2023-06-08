# Unnecessary subtraction to make variables as zero

`withdrawalAmount` is equal to `lotItem.bids[msg.sender]` on line 125.
Then, in line 128, it subtracts `withdrawalAmount` from `lotItem.bids[msg.sender]`, which is effectively making itself zero.

This saves gas too.

```
File: contracts/Auction.sol
125:        uint256 withdrawalAmount = lotItem.bids[msg.sender];
126:        if (withdrawalAmount == 0) revert InSufficientETH();
127:
128: -----> lotItem.bids[msg.sender] -= withdrawalAmount;
```

Link: https://github.com/code-423n4/2023-06-stader/blob/d5f7854fdf70547c6476c00be5d97c85f2c8d064/contracts/Auction.sol#L128

##### Fix:

```
-        lotItem.bids[msg.sender] -= withdrawalAmount;
+        lotItem.bids[msg.sender] = 0;
```

Other instances:

```
File: contracts/OperatorRewardsCollector.sol

48:        uint256 amount = balances[operator];
49:        balances[operator] -= amount;
```

Link: https://github.com/code-423n4/2023-06-stader/blob/d5f7854fdf70547c6476c00be5d97c85f2c8d064/contracts/OperatorRewardsCollector.sol#L48-L49

---

# Possible Infinite Loops

If the condition triggers the `continue`, then the loop variable does not get incremented. The condition never changes, as the same condition is checked over and over again, resulting in an infinite loop.


```
File: contracts/PermissionedNodeRegistry.sol

207:            if (!operatorStructById[i].active) {
208:                continue;
209:            }
```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L207-L209

##### Fix: Reverse the condition and put everything inside the if block. Keep i++ outside the if block.

```
-                if (!operatorStructById[i].active) {
-                    continue;
+                if (operatorStructById[i].active) {
+                    remainingOperatorCapacity[i] = getOperatorQueuedValidatorCount(i);
+                    selectedOperatorCapacity[i] = Math.min(remainingOperatorCapacity[i], validatorPerOperator);
+                    totalValidatorToDeposit += selectedOperatorCapacity[i];
+                    remainingOperatorCapacity[i] -= selectedOperatorCapacity[i];
                 }
-                remainingOperatorCapacity[i] = getOperatorQueuedValidatorCount(i);
-                selectedOperatorCapacity[i] = Math.min(remainingOperatorCapacity[i], validatorPerOperator);
-                totalValidatorToDeposit += selectedOperatorCapacity[i];
-                remainingOperatorCapacity[i] -= selectedOperatorCapacity[i];
                 unchecked {
                     ++i;
                 }
```

Other instances:

```
File: contracts/PermissionedNodeRegistry.sol

227:                if (!operatorStructById[i].active) {
228:                    continue;
229:                }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L227-L229

---

# Several State variables can be initialized outside the initialize method

There are many state variables that are initialized in the `initialize` method with a constant. These variables have corresponding methods to update them as well.

Initializing them in the `initialize` method seems unnecessary, as thats the first step after a contract is deployed.

As long as the variables have a setter method, and they are initialized to a constant value, it can be moved out from the `initialize` method. This will also save a lot of gas.

```
File: contracts/Penalty.sol

31:    function initialize(
32:        address _admin,
33:        address _staderConfig,
34:        address _ratedOracleAddress
35:    ) external initializer {
        ...
        ...
44:        mevTheftPenaltyPerStrike = 1 ether;
45:        missedAttestationPenaltyPerStrike = 0.2 ether;
46:        validatorExitPenaltyThreshold = 2 ether;
        ...
        ...
50:    }

```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L44-46

##### Fix:

```
-    uint256 public override mevTheftPenaltyPerStrike;
-    uint256 public override missedAttestationPenaltyPerStrike;
-    uint256 public override validatorExitPenaltyThreshold;
+    uint256 public mevTheftPenaltyPerStrike = 1 ether;
+    uint256 public missedAttestationPenaltyPerStrike = 0.2 ether;
+    uint256 public validatorExitPenaltyThreshold = 2 ether;
...
...
    function initialize(
        address _admin,
        address _staderConfig,
        address _ratedOracleAddress
    ) external initializer {
         ...
         ...
-        mevTheftPenaltyPerStrike = 1 ether;
-        missedAttestationPenaltyPerStrike = 0.2 ether;
-        validatorExitPenaltyThreshold = 2 ether;
         ...
         ...
    }

```

Other Instances:

```
File: contracts/Auction.sol

38:        duration = 2 * MIN_AUCTION_DURATION;
39:        bidIncrement = 5e15;
40:        nextLot = 1;
```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L38-L40


```
File: contracts/PermissionedNodeRegistry.sol

73:        nextOperatorId = 1;
74:        nextValidatorId = 1;
75:        operatorIdForExcessDeposit = 1;
76:        inputKeyCountLimit = 50;
77:        maxOperatorId = 10;
78:        maxNonTerminalKeyPerOperator = 50;
79:        verifiedKeyBatchSize = 50;
```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L73-L79

```
File: contracts/PermissionedPool.sol

45:        protocolFee = 500;
46:        operatorFee = 500;
```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L45-L46

```
File: contracts/PermissionlessNodeRegistry.sol

73:        nextOperatorId = 1;
74:        nextValidatorId = 1;
75:        inputKeyCountLimit = 30;
76:        maxNonTerminalKeyPerOperator = 50;
77:        verifiedKeyBatchSize = 50;
```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L73-L77

```
File: contracts/PermissionlessPool.sol

43:        protocolFee = 500;
44:        operatorFee = 500;
```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L43-L44

```
File: contracts/PoolSelector.sol

40:        poolAllocationMaxSize = 50;
```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L40

```
File: contracts/StaderOracle.sol

69:        erChangeLimit = 500; //5% deviation threshold
```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L69

```
File: contracts/StaderStakePoolsManager.sol

57:        excessETHDepositCoolDown = 3 * 7200;
```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L57

```
File: contracts/UserWithdrawalManager.sol

61:        nextRequestIdToFinalize = 1;
62:        nextRequestId = 1;
63:        finalizationBatchLimit = 50;
64:        maxNonRedeemedUserRequestCount = 1000;
```

Link: https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L61-L64

---
