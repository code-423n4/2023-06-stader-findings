# Issue: Incomplete Validation (QA)

## Description

In the `addBid` function, the function should also check if `block.number < lotItem.endBlock` case. The current code stack doesn’t use `addBid` function yet, but lacking this check may lead more serious issues.

## Location
contracts/Auction.sol#L67

## Suggestion

Adding the check `block.number < lotItem.endBlock`

# Issue: Low-level call (QA)
## Description of the security issue

The use of low-level calls such as `address.call()` (in this case hidden inside UtilLib.sendValue()) can potentially introduce security issues and could make your contract less safe if not used properly. These low-level calls also bypass any checks for function visibility or argument matching, which can lead to unexpected behavior.

## Location of the security issue

The use of UtilLib.sendValue() within the withdraw function.

## How to resolve the security issue: 

Instead of low-level calls, consider using Solidity's built-in high-level address methods for transferring ether. In this case, the safer way to transfer ether would be using the transfer method. Here's how you can implement it:
``
payable(staderConfig.getStaderTreasury()).transfer(protocolShare);
``

# Issue: DoS with Unexpected Revert (QA)

## Description: 

The `updateTotalPenaltyAmount` function may revert unexpectedly due to revert `ValidatorSettled()`; if the `getValidatorSettleStatus` returns true. In a loop, an unexpected revert in one operation can lead to denial of service as the rest of the operations won't be performed.

## Location:
``
function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
    uint256 reportedValidatorCount = _pubkey.length;
    for (uint256 i; i < reportedValidatorCount; ) {
        if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
            revert ValidatorSettled();
        }
        //rest of the code
        unchecked {
            ++i;
        }
    }
}
``

## Resolution

Instead of using revert, log an event or skip the validator for which getValidatorSettleStatus returns true.
``
function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
    uint256 reportedValidatorCount = _pubkey.length;
    for (uint256 i; i < reportedValidatorCount; ) {
        if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
            emit ValidatorSettled(_pubkey[i]);
        } else {
            //rest of the code
        }
        unchecked {
            ++i;
        }
    }
}
``

# Issue: Lack of Checking `_nodeOperator` (QA)

## Description
In the function `getOperatorTotalNonTerminalKeys`, there should be a check to see if the `_nodeOperator` is a valid operator.

## Location
contracts/PermissionedNodeRegistry.sol#L522-L544

## Suggestion
Adding check to validate the input `_nodeOperator`

# Issue: Lack of checking `_pageSize` (QA)

## Description
In the `getAllActiveValidators` function, it would be safer to check that _pageSize is greater than 0.

## Location
contracts/PermissionedNodeRegistry.sol#L584

## Suggestion
Add checking `_pageSize > 0`

# Issue name: Lack of Input Validation

## Description of the security issue: 

The `checkInputKeysCountAndCollateral` function could potentially accept zero as a valid `_operatorId`. While the function does check if `_pubkeyLength`, `_preDepositSignatureLength`, and `_depositSignatureLength` are equal, it doesn't check if they're greater than zero. Also the `getOperatorTotalKeys` function also doesn’t check the input _operatorId.

## Location of the security issue with code snippet:
contracts/PermissionedNodeRegistry.sol#L687-L717

## How to resolve the security issue with code example: 
Add an additional validation check for `_operatorId` to be non-zero for all those functions that lack this check. 

# Issue name: Potential Underflow in decreaseTotalActiveValidatorCount

## Description of the security issue: 

The `decreaseTotalActiveValidatorCount` function subtracts `_count` from `totalActiveValidatorCount `without ensuring that `_count` is less than or equal to totalActiveValidatorCount. If `_count` is greater than totalActiveValidatorCount, this operation will underflow and the totalActiveValidatorCount will be set to a very large number.

## Location
contracts/PermissionedNodeRegistry.sol#L747-L750

## How to resolve the security issue with code example: 
Add a require statement that `ensures _count` is less than or equal to `totalActiveValidatorCount` before subtracting.

```
function decreaseTotalActiveValidatorCount(uint256 _count) internal {
    require(_count <= totalActiveValidatorCount, "Count is greater than total");
    totalActiveValidatorCount -= _count;
    emit DecreasedTotalActiveValidatorCount(totalActiveValidatorCount);
}
```

# Issue name: Missing function visibility (QA)

## Description of the security issue: 

In the `increasePreDepositValidatorCount` and `decreasePreDepositValidatorCount` functions, visibility is not explicitly declared. By default, these functions are public, which could potentially allow unauthorized access to modify the preDepositValidatorCount state variable.

## How to resolve the security issue with code example: 

It is good practice to explicitly declare the visibility of your functions. These functions should be declared as internal if they are not meant to be accessed outside of this contract.

# Issue: Missing Validation for `_validatorId`

## Description
In the `slashValidatorSD` function, the input `_validatorId` is not checked. 

## Suggestion
Adding the check for `_validatorId` in the `slashValidatorSD` function

# Issue Name: Potential Underflow in Claim Function (QA)

## Description of the Security Issue: 

In the `claim` function, there is a potential for underflow when subtracting the claimed amount from the total remaining rewards. There are no checks in place to ensure that the claimed amount is less than or equal to the total remaining rewards.

## Location of the security issue with code snippet:
```
function claim(
    uint256[] calldata _index,
    uint256[] calldata _amountSD,
    uint256[] calldata _amountETH,
    bytes32[][] calldata _merkleProof
) external override nonReentrant whenNotPaused {
    // ...

    if (totalAmountETH > 0) {
        totalOperatorETHRewardsRemaining -= totalAmountETH;
        // ...
    }

    if (totalAmountSD > 0) {
        totalOperatorSDRewardsRemaining -= totalAmountSD;
        // ...
    }

    // ...
}
```
## How to resolve the security issue with code example: 

Before subtracting the claimed amount, check if the claimed amount is less than or equal to the remaining rewards. If it is not, revert the transaction.
```
function claim(
    uint256[] calldata _index,
    uint256[] calldata _amountSD,
    uint256[] calldata _amountETH,
    bytes32[][] calldata _merkleProof
) external override nonReentrant whenNotPaused {
    // ...

    if (totalAmountETH > 0) {
        require(totalOperatorETHRewardsRemaining >= totalAmountETH, "Not enough ETH rewards remaining");
        totalOperatorETHRewardsRemaining -= totalAmountETH;
        // ...
    }

    if (totalAmountSD > 0) {
        require(totalOperatorSDRewardsRemaining >= totalAmountSD, "Not enough SD rewards remaining");
        totalOperatorSDRewardsRemaining -= totalAmountSD;
        // ...
    }

    // ...
}
```

# Issue Name: Potential Reentrancy Attack (QA)

## Description of the Security Issue: 

The contract uses a `nonReentrant` modifier from OpenZeppelin library to prevent reentrancy attacks, which is a best practice. However, it is crucial to note that `nonReentrant` modifier is not applied to all public and external functions. If any external call can call back into any function without the modifier, it might potentially allow reentrancy attacks.

## Location of the security issue with code snippet:

```solidity
receive() external payable {
    emit ETHReceived(msg.sender, msg.value);
}
```

## How to resolve the security issue with code example: 

Apply the `nonReentrant` modifier to any function that makes external calls and ensure that external calls are the last actions performed in the function. If you can't make the external call the last operation, you should ensure that the contract's state is not altered after the external call.

```solidity
receive() external payable nonReentrant {
    emit ETHReceived(msg.sender, msg.value);
}
```

NOTE: Although the `receive` function seems benign, the nonReentrant modifier still could be applied to adhere to best practices.

# Issue name: No Input Sanitation (QA)

## Description: 
Most of the update functions lack input sanitation checks. It's generally recommended to validate user inputs before processing them.

## Location: 
In functions like `updateSocializingPoolCycleDuration()`, `updateRewardsThreshold()`, etc.

## How to resolve: Add appropriate checks for input parameters.
```
function updateSocializingPoolCycleDuration(uint256 _socializingPoolCycleDuration) external onlyRole(MANAGER) {
    require(_socializingPoolCycleDuration > 0, "Socializing Pool Cycle Duration must be greater than zero");
    setVariable(SOCIALIZING_POOL_CYCLE_DURATION, _socializingPoolCycleDuration);
}
```
# Issue name: Ambiguous error messages (QA)

## Description: 
The function `verifyDepositAndWithdrawLimits()` reverts with "InvalidLimits" without specifically mentioning which condition failed.

## How to resolve: 

Use specific error messages for each failed condition to make debugging easier.
```
function verifyDepositAndWithdrawLimits() internal view {
    require(variablesMap[MIN_DEPOSIT_AMOUNT] != 0, "Minimum deposit amount cannot be zero");
    require(variablesMap[MIN_WITHDRAW_AMOUNT] != 0, "Minimum withdrawal amount cannot be zero");
    require(variablesMap[MIN_DEPOSIT_AMOUNT] <= variablesMap[MAX_DEPOSIT_AMOUNT], "Minimum deposit amount exceeds maximum");
    require(variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MAX_WITHDRAW_AMOUNT], "Minimum withdrawal amount exceeds maximum");
    require(variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MIN_DEPOSIT_AMOUNT], "Minimum withdrawal amount exceeds minimum deposit amount");
    require(variablesMap[MAX_WITHDRAW_AMOUNT] >= variablesMap[MAX_DEPOSIT_AMOUNT], "Maximum withdrawal amount is less than maximum deposit amount");
}
```

# Issue name: Arbitrary ETH Withdrawal (QA)

## Description: 

The function withdrawFund() allows a 'MANAGER' role to withdraw any amount of ETH. This could potentially allow for all funds in the contract to be withdrawn by a malicious actor who gains access to a 'MANAGER' account.
Proof of Concept: If a bad actor gains access to a 'MANAGER' account, they can call the function withdrawFund() with _amount set to the entire balance of the contract.

## Location:
```
function withdrawFund(uint256 _amount) external override nonReentrant {
    UtilLib.onlyManagerRole(msg.sender, staderConfig);
    if (address(this).balance < _amount || _amount == 0) {
        revert InvalidAmountProvided();
    }
    (bool success, ) = payable(msg.sender).call{value: _amount}('');
    if (!success) {
        revert TransferFailed();
    }
    emit FundWithdrawn(_amount);
}
```

## How to resolve: One possible solution is to set a limit on the amount of ETH that can be withdrawn at one time. This limit could be a percentage of the total funds held by the contract, thus preventing a 'MANAGER' from draining all funds at once.

```
uint256 withdrawalLimit = address(this).balance / 10; // allow withdrawal of up to 10% of the balance at a time

function withdrawFund(uint256 _amount) external override nonReentrant {
    UtilLib.onlyManagerRole(msg.sender, staderConfig);
    require(_amount <= withdrawalLimit, "Withdrawal amount exceeds limit");
    if (address(this).balance < _amount || _amount == 0) {
        revert InvalidAmountProvided();
    }
    (bool success, ) = payable(msg.sender).call{value: _amount}('');
    if (!success) {
        revert TransferFailed();
    }
    emit FundWithdrawn(_amount);
}
```

# Issue name: Missing check for zero amount in reimburseUserFund (QA)

## Description: 

The `reimburseUserFund` function allows transferring zero funds. This could lead to unnecessary gas fees and misleading event emissions.

## Location:
```
function reimburseUserFund(uint256 _amount) external override nonReentrant {
    UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());
    if (address(this).balance < _amount) {
        revert InSufficientBalance();
    }
    IPermissionedPool(staderConfig.getPermissionedPool()).receiveInsuranceFund{value: _amount}();
}
```
## How to resolve: 
Add a `require` statement to validate that `_amount` is greater than zero.

```
function reimburseUserFund(uint256 _amount) external override nonReentrant {
    require(_amount > 0, "Reimbursement amount must be greater than zero");
    UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());
    if (address(this).balance < _amount) {
        revert InSufficientBalance();
    }
    IPermissionedPool(staderConfig.getPermissionedPool()).receiveInsuranceFund{value: _amount}();
}
```

# Issue Name: Use of Block Timestamp (QA)

## Description: 

Solidity's `block.timestamp` can be manipulated by miners within a certain range (+/- 900 seconds). It is not recommended to use block.timestamp in cases where exact times are critical for the contract's operation.

## Location
emit ExchangeRateSubmitted(
         msg.sender,
         _exchangeRate.reportingBlockNumber,
         _exchangeRate.totalETHBalance,
         _exchangeRate.totalETHXSupply,
         block.timestamp // This can be manipulated by miners
);

## How to resolve: 
Instead of relying on block.timestamp, use block.number as a measure of time. Block number is a safer alternative as it can't be manipulated by miners.

```
emit ExchangeRateSubmitted(
         msg.sender,
         _exchangeRate.reportingBlockNumber,
         _exchangeRate.totalETHBalance,
         _exchangeRate.totalETHXSupply,
         block.number // Replaced block.timestamp with block.number
);
```

# Issue Name: Use of tx.origin (QA)

## Description of the security issue: 

The use of `tx.origin` can lead to security vulnerabilities, as it refers to the original address that initiated the transaction and not necessarily the contract that called the function. This could be exploited in certain cases.

## Location of the security issue with code snippet:
```
function depositEth(uint256 _gasPrice) external override payable returns (uint256) {
    ...
    bytes32 ethDepositRequestId = keccak256(abi.encodePacked(tx.origin, block.number, address(this)));
    ...
}
```

## How to resolve the security issue with code example:
Use msg.sender instead of tx.origin for more secure behavior.
```
function depositEth(uint256 _gasPrice) external override payable returns (uint256) {
    ...
    bytes32 ethDepositRequestId = keccak256(abi.encodePacked(msg.sender, block.number, address(this)));
    ...
}
```

# Issue Name: No Input Validation on receiveExcessEthFromPool (QA)

## Description: 
The function `receiveExcessEthFromPool` does not have any validation for the input and does not restrict the sender. It allows anyone to call the function and emit an event with an arbitrary pool ID and amount of ether. Although this function may not directly affect the state, it can be exploited to emit fake events.

## Location of the security issue with code snippet:

```
function receiveExcessEthFromPool(uint8 _poolId) external payable override {
    emit ReceivedExcessEthFromPool(_poolId);
}
```
## How to resolve the security issue:
You should add access control mechanisms to the function to ensure only authorized accounts can call it. Also, validate the `_poolId` if necessary.

```
function receiveExcessEthFromPool(uint8 _poolId) external payable override onlyAuthorized {
    require(isValidPoolId(_poolId), "Invalid pool ID");
    emit ReceivedExcessEthFromPool(_poolId);
}
```

# Issue Name: Lack of Input Validation for Deploy Functions (QA)

## Description: 
The `deployWithdrawVault` and `deployNodeELRewardVault` functions accept inputs that are used for deploying new vaults. However, there is no validation of these inputs, which might lead to unintended behavior. It seems multiple functions in `VaultFactor.sol` lacking input validation. 

## Location of the security issue:
```
function deployWithdrawVault(
    uint8 _poolId,
    uint256 _operatorId,
    uint256 _validatorCount,
    uint256 _validatorId
) public override onlyRole(NODE_REGISTRY_CONTRACT) returns (address) {
    ...
}
```
## How to resolve the security issue:
Add input validation to ensure that inputs are within expected ranges or meet certain conditions before proceeding.

```
function deployWithdrawVault(
    uint8 _poolId,
    uint256 _operatorId,
    uint256 _validatorCount,
    uint256 _validatorId
) public override onlyRole(NODE_REGISTRY_CONTRACT) returns (address) {
    require(_poolId > 0, "Pool ID should be greater than 0");
    require(_operatorId > 0, "Operator ID should be greater than 0");
    require(_validatorCount > 0, "Validator count should be greater than 0");
    require(_validatorId > 0, "Validator ID should be greater than 0");

    ...
}
```












