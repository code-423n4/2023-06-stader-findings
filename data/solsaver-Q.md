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