
## Summary 

### Gas Optimization

|no |Issue|Instances|
|----|-------|------|
| [G-01] | Caching global variables is expensive than using the variable itself. | 2 | - | 
| [G-02] | Emitting storage values instead of the memory one. | 5 | - | 
| [G-03] | Amounts should be checked for 0 before calling a transfer. | 3 | - | 
| [G-04] | require() or revert() statements that check input arguments should be at the top of the function (Also restructured some if’s). | 3 | - | 
| [G-05] | use nestead if instead of && to save gas. | 7 | - | 
| [G-06] | Do not calculate constants to save gas. | 1 | - | 
| [G-07] | Use hardcode address instead address(this) | 7 | - | 
| [G-08] | Use calldata instead of memory for function arguments that do not get mutated | 2 | - | 
| [G-09] | Gas overflow during iteration (DoS) | 1 | - | 
| [G-10] | Empty blocks should be removed or emit something | 3 | - | 
| [G-11] | public functions not called by the contract should be declared external instead | 9 | - | 
| [G-12] | Use != 0 instead of > 0 for unsigned integer comparison to save gas | 5 | - | 
 | 



### [G-01] Caching global variables is expensive than using the variable itself.
in the claim() function, caching the msg.sender address in a local variable is not necessary, as msg.sender is only read once and is not used subsequently in the function.
Therefore, you can simplify the code and reduce gas costs by removing the operator variable and modifying the balances[operator] -= amount; line to balances[msg.sender] = 0;, like this:

# Don’t cache msg.sender in the following 
```solidity
file:    OperatorRewardsCollector.sol
46       function claim() external whenNotPaused {
        address operator = msg.sender;
        uint256 amount = balances[operator];
        balances[operator] -= amount;

        address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig);
        UtilLib.sendValue(operatorRewardsAddr, amount);
        emit Claimed(operatorRewardsAddr, amount);
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#LL46C1-L54

## Recommaded code:

```solidity
function claim() external whenNotPaused {
    uint256 amount = balances[msg.sender];
    balances[msg.sender] = 0;

    address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig);
    UtilLib.sendValue(operatorRewardsAddr, amount);
    emit Claimed(operatorRewardsAddr, amount);
}
```
This code reads the balances[msg.sender] value into the local variable amount, sets balances[msg.sender] to zero, and then uses amount to transfer the balance to the operator rewards address. By removing the unnecessary operator variable, the code is simplified and the gas cost is reduced.

# Don’t cache msg.sender in the following 

 in the depositSDAsCollateral() function, caching the msg.sender address in a local variable is not necessary, as msg.sender is only read once and is not used subsequently in the function.

Therefore, you can simplify the code and reduce gas costs by removing the operator variable and modifying the operatorSDBalance[operator] += _sdAmount; line to operatorSDBalance[msg.sender] += _sdAmount;, like this:

```solidity
file:   SDCollateral.sol
43      function depositSDAsCollateral(uint256 _sdAmount) external override {
        address operator = msg.sender;
        operatorSDBalance[operator] += _sdAmount;

        if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }

        emit SDDeposited(operator, _sdAmount);
    }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#LL43C1-L52

## Recommaded code:

```solidity
function depositSDAsCollateral(uint256 _sdAmount) external override {
    operatorSDBalance[msg.sender] += _sdAmount;

    if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
        revert SDTransferFailed();
    }

    emit SDDeposited(msg.sender, _sdAmount);
}
```
This code adds _sdAmount to the operatorSDBalance mapping for msg.sender, transfers the SD tokens from msg.sender to the contract, and emits an event. By removing the unnecessary operator variable, the code is simplified and the gas cost is reduced.





### [G-02] Emitting storage values instead of the memory one.

In Solidity, it's generally more efficient to emit event values from memory variables instead of storage variables. Emitting storage values can result in higher gas costs and reduced efficiency of the smart contract.

When emitting an event, it's best to use local memory variables to store values that will be emitted. By doing so, we can avoid the higher gas costs associated with accessing storage variables.

For example, suppose we have a contract with a private state variable myValue that we want to emit whenever it changes. Instead of emitting the value of myValue directly, we should first assign it to a local memory variable and then emit the local variable. This approach is shown below:

```solidity
contract MyContract {
    uint256 private myValue;

    event ValueChanged(uint256 newValue);

    function setValue(uint256 _newValue) public {
        myValue = _newValue;
        emit ValueChanged(_newValue);
    }
}

```
In this example, we use a local memory variable _newValue to store the new value of myValue and then emit the value from the memory variable. This approach is more efficient than emitting the value of myValue directly from storage.

## Emit this _owner instead of owner

```solidity
file:   VaultProxy.sol
68      function updateOwner(address _owner) external override onlyOwner {
        UtilLib.checkNonZeroAddress(_owner);
        owner = _owner;
        emit UpdatedOwner(owner);
    }

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#LL68C1-L72

## Recommaded code:

```solidity
  function updateOwner(address _owner) external override onlyOwner {
        UtilLib.checkNonZeroAddress(_owner);
        owner = _owner;
    -   emit UpdatedOwner(owner);

    +   emit UpdatedOwner(_owner);
    }

```


## Emit this _duration instead of duration
```solidit
file:   Auction.sol
144     function updateDuration(uint256 _duration) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        if (_duration < MIN_AUCTION_DURATION) revert ShortDuration();
        duration = _duration;
        emit AuctionDurationUpdated(duration);
    }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#LL144C1-L149

## Recommaded code:

```solidity
 function updateDuration(uint256 _duration) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        if (_duration < MIN_AUCTION_DURATION) revert ShortDuration();
        duration = _duration;
    -   emit AuctionDurationUpdated(duration);

    +   emit AuctionDurationUpdated(_duration);
    }

```

## Emit this _nextQueuedValidatorIndex instead of nextQueuedValidatorIndex

```solidity
file:   PermissionlessNodeRegistry.sol

268     function updateNextQueuedValidatorIndex(uint256 _nextQueuedValidatorIndex) external {
        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_POOL());
        nextQueuedValidatorIndex = _nextQueuedValidatorIndex;
        emit UpdatedNextQueuedValidatorIndex(nextQueuedValidatorIndex);
    }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#LL268C1-L272

## Recommaded code:

```solidity

 function updateNextQueuedValidatorIndex(uint256 _nextQueuedValidatorIndex) external {
        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_POOL());
        nextQueuedValidatorIndex = _nextQueuedValidatorIndex;

    -   emit UpdatedNextQueuedValidatorIndex(nextQueuedValidatorIndex);

    +   emit UpdatedNextQueuedValidatorIndex(_nextQueuedValidatorIndex);
    }


```
## Emit this _inputKeyCountLimit instead of inputKeyCountLimit

```solidity
file:   PermissionedNodeRegistry.sol
427     function updateInputKeyCountLimit(uint16 _inputKeyCountLimit) external override {
        UtilLib.onlyOperatorRole(msg.sender, staderConfig);
        inputKeyCountLimit = _inputKeyCountLimit;
        emit UpdatedInputKeyCountLimit(inputKeyCountLimit);
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#LL427C1-L431

## Recommaded code:

```solidity
        function updateInputKeyCountLimit(uint16 _inputKeyCountLimit) external override {
        UtilLib.onlyOperatorRole(msg.sender, staderConfig);
        inputKeyCountLimit = _inputKeyCountLimit;
    -   emit UpdatedInputKeyCountLimit(inputKeyCountLimit);

    +   emit UpdatedInputKeyCountLimit(_inputKeyCountLimit);
    }

```

using a local variable _count instead of accessing the state variable totalActiveValidatorCount directly can potentially save gas cost in the decreaseTotalActiveValidatorCount function.
By using a local variable, we avoid the extra gas cost of loading the state variable from storage, which can be expensive in terms of gas usage.

```solidity
file: 
747    function decreaseTotalActiveValidatorCount(uint256 _count) internal {
        totalActiveValidatorCount -= _count;
        emit DecreasedTotalActiveValidatorCount(totalActiveValidatorCount);
    }
```
## Recommaded code:

```solidity

function decreaseTotalActiveValidatorCount(uint256 _count) internal {
    uint256 newTotalActiveValidatorCount = totalActiveValidatorCount - _count;
    totalActiveValidatorCount = newTotalActiveValidatorCount;
    emit DecreasedTotalActiveValidatorCount(newTotalActiveValidatorCount);
}
```
In this implementation, we subtract the _count parameter from totalActiveValidatorCount and store the result in the local variable newTotalActiveValidatorCount. We then update the value of totalActiveValidatorCount with the new value and emit an event with the new value.

By using a local variable instead of accessing the state variable directly, we can potentially save gas cost, especially if this function is called frequently in the smart contract.


### [G-03]  Amounts should be checked for 0 before calling a transfer

In Solidity, the transfer() function is commonly used to transfer ether or tokens from one address to another. When calling the transfer() function, it is important to check the amount being transferred to ensure that it is greater than 0, as transferring 0 amounts is not necessary and can consume unnecessary gas.

The statement "Amounts should be checked for 0 before calling a transfer to save gas" means that checking the amount being transferred before calling the transfer() function can help to reduce the gas cost of the contract and optimize the Solidity code.

For example, consider the following code:

```solidity
contract MyContract {
  address payable public myAddress;
  
  function send(uint256 _amount) public {
    require(_amount > 0, "Amount must be greater than 0");
    myAddress.transfer(_amount);
  }
}
```
This code checks whether _amount is greater than 0 using the require() function before calling the transfer() function. If _amount is not greater than 0, the function will revert with an error message, preventing a transfer of 0 amounts and reducing the gas cost of the contract.

Therefore, checking the amount being transferred before calling the transfer() function can help to reduce the gas cost of the contract and optimize the Solidity code.

```solidity
file:  Auction.sol
48      function createLot(uint256 _sdAmount) external override whenNotPaused {
        lots[nextLot].startBlock = block.number;
        lots[nextLot].endBlock = block.number + duration;
        lots[nextLot].sdAmount = _sdAmount;

        LotItem storage lotItem = lots[nextLot];

        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }
        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
        nextLot++;
    }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#LL48C1-L60


### Recommanded code:

```solidity    
        function createLot(uint256 _sdAmount) external override whenNotPaused {
        lots[nextLot].startBlock = block.number;
        lots[nextLot].endBlock = block.number + duration;
        lots[nextLot].sdAmount = _sdAmount;

        LotItem storage lotItem = lots[nextLot];
        
    +    require(_sdAmount > 0, "Amount must be greater than 0");

        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }
        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
        nextLot++;
    }
```

```solidity
file:    SDCollateral.sol
58       function withdraw(uint256 _requestedSD) external override {
        address operator = msg.sender;
        uint256 opSDBalance = operatorSDBalance[operator];

        if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
            revert InsufficientSDToWithdraw(opSDBalance);
        }
        operatorSDBalance[operator] -= _requestedSD;

        // cannot use safeERC20 as this contract is an upgradeable contract
        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
            revert SDTransferFailed();
        }

        emit SDWithdrawn(operator, _requestedSD);
    }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#LL58C1-L73


### Recommanded code:

```solidity
       function withdraw(uint256 _requestedSD) external override {
        address operator = msg.sender;
        uint256 opSDBalance = operatorSDBalance[operator];

        if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
            revert InsufficientSDToWithdraw(opSDBalance);
        }
        operatorSDBalance[operator] -= _requestedSD;


    +   require(_requestedSD > 0, "Amount must be greater than 0");   

        // cannot use safeERC20 as this contract is an upgradeable contract
        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
            revert SDTransferFailed();
        }

        emit SDWithdrawn(operator, _requestedSD);
    }


```


```solidity
file:   PermissionlessNodeRegistry.sol
392     function transferCollateralToPool(uint256 _amount) external override nonReentrant {
        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_POOL());
        IPermissionlessPool(staderConfig.getPermissionlessPool()).receiveRemainingCollateralETH{value: _amount}();
        emit TransferredCollateralToPool(_amount);
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#LL392C5-L396

### Recommanded code:

```solidity

   function transferCollateralToPool(uint256 _amount) external override nonReentrant {
        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_POOL());

    +   require(_amount > 0, "Amount must be greater than 0");     

        IPermissionlessPool(staderConfig.getPermissionlessPool()).receiveRemainingCollateralETH{value: _amount}();
        emit TransferredCollateralToPool(_amount);
    }
```




## [G-04] require() or revert() statements that check input arguments should be at the top of the function (Also restructured some if’s)
Declaring require() or revert() statements that check input arguments at the top of the function can save gas because it allows the function to fail early and cheaply.

When a require() or revert() statement is executed, any remaining gas in the current transaction is refunded to the user. This means that if a require() or revert() statement is executed early in the function, before any state changes or expensive calculations are performed, the gas used by the function will be minimal and the remaining gas refunded to the user.

By declaring require() or revert() statements that check input arguments at the top of the function, we ensure that any invalid inputs are caught early and the function fails before any expensive calculations or state changes are performed. This saves gas and reduces the risk of running out of gas during execution.

Additionally, by structuring if statements in a way that places constant checks before state variable checks, we can further optimize gas usage. This is because constant checks can be performed more efficiently than state variable checks.

Overall, placing require() or revert() statements that check input arguments at the top of the function and structuring if statements to place constant checks before state variable checks can help to optimize gas usage and ensure that functions fail early and cheaply.


```solidity
file:  Auction.sol

48    function createLot(uint256 _sdAmount) external override whenNotPaused {
        lots[nextLot].startBlock = block.number;
        lots[nextLot].endBlock = block.number + duration;
        lots[nextLot].sdAmount = _sdAmount;

        LotItem storage lotItem = lots[nextLot];

        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }
        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
        nextLot++;
    }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#LL48C1-L60

### Recommanded code:

```solidity

  function createLot(uint256 _sdAmount) external override whenNotPaused {
    // Check that the input argument is valid
+    require(_sdAmount > 0, "Invalid _sdAmount");

    // Get the address of the Stader token contract
    address staderTokenAddress = staderConfig.getStaderToken();

    // Transfer the specified SD amount from the caller to this contract
    if (!IERC20(staderTokenAddress).transferFrom(msg.sender, address(this), _sdAmount)) {
        revert SDTransferFailed();
    }

    // Create a new lot with the specified parameters
    LotItem storage lotItem = lots[nextLot];
    lotItem.startBlock = block.number;
    lotItem.endBlock = block.number + duration;
    lotItem.sdAmount = _sdAmount;

    // Emit an event to signal that a new lot has been created
    emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);

    // Increment the lot counter for the next lot
    nextLot++;
}


```

```solidity
file:   Auction.sol

93    function transferHighestBidToSSPM(uint256 lotId) external override nonReentrant {
        LotItem storage lotItem = lots[lotId];
        uint256 ethAmount = lotItem.highestBidAmount;

        if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
        if (ethAmount == 0) revert NoBidPlaced();
        if (lotItem.ethExtracted) revert AlreadyClaimed();

        lotItem.ethExtracted = true;
        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveEthFromAuction{value: ethAmount}();
        emit ETHClaimed(lotId, staderConfig.getStakePoolManager(), ethAmount);
    }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#LL93C1-L104

## Recommanded code: 

```solidity
function transferHighestBidToSSPM(uint256 lotId) external override nonReentrant {
    // Get the lot item for the specified lot ID
    LotItem storage lotItem = lots[lotId];

    // Check that the auction has ended and a bid has been placed
    require(block.number > lotItem.endBlock, "Auction not ended");
    require(lotItem.highestBidAmount > 0, "No bid placed");

    // Check that the ETH for this lot has not been claimed before
    require(!lotItem.ethExtracted, "Already claimed");

    // Mark the ETH for this lot as claimed
    lotItem.ethExtracted = true;

    // Transfer the ETH to the Stader Stake Pool Manager contract
    IStaderStakePoolManager stakePoolManager = IStaderStakePoolManager(staderConfig.getStakePoolManager());
    stakePoolManager.receiveEthFromAuction{value: lotItem.highestBidAmount}();

    // Emit an event to signal that the ETH has been claimed
    emit ETHClaimed(lotId, staderConfig.getStakePoolManager(), lotItem.highestBidAmount);
}

```

```solidity
file:Penalty.sol

98    function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
        uint256 reportedValidatorCount = _pubkey.length;
        for (uint256 i; i < reportedValidatorCount; ) {
            if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
                revert ValidatorSettled();
            }
            bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
            // Retrieve the penalty for changing the fee recipient address based on Rated.network data.
            uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
            uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);

            // Compute the total penalty for the given validator public key,
            // taking into account additional penalties and penalty reversals from the DAO.
            uint256 totalPenalty = _mevTheftPenalty + _missedAttestationPenalty + additionalPenaltyAmount[pubkeyRoot];
            totalPenaltyAmount[_pubkey[i]] = totalPenalty;
            if (totalPenalty >= validatorExitPenaltyThreshold) {
                emit ForceExitValidator(_pubkey[i]);
            }
            unchecked {
                ++i;
            }
        }
    }

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#LL98C1-L120


## Recommanded code:

```solidity

function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
    // Check that the input array is not empty
    require(_pubkey.length > 0, "Empty pubkey array");

    for (uint256 i = 0; i < _pubkey.length; i++) {
        // Check that the validator has not already settled
        if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
            revert ValidatorSettled();
        }

        // Calculate the MEV theft penalty and missed attestation penalty
        bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
        uint256 mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
        uint256 missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);

        // Compute the total penalty for the given validator public key,
        // taking into account additional penalties and penalty reversals from the DAO.
        uint256 totalPenalty = mevTheftPenalty + missedAttestationPenalty + additionalPenaltyAmount[pubkeyRoot];
        totalPenaltyAmount[_pubkey[i]] = totalPenalty;

        // Emit an event if the total penalty is greater than or equal to the validator exit penalty threshold
        if (totalPenalty >= validatorExitPenaltyThreshold) {
            emit ForceExitValidator(_pubkey[i]);
        }
    }
}

```



### [G-05] use nestead if instead of && to save gas
Using nested if statements instead of the && operator can sometimes save gas in smart contract code.
The reason for this is that the && operator performs a single logical conjunction operation between two expressions, which can be relatively expensive in terms of gas cost. In contrast, using nested if statements can avoid the need for a logical conjunction operation, which can result in gas savings.

```solidity
file:     SocializingPool.sol

164       if (_amountSD[i] == 0 && _amountETH[i] == 0) 

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#LL146C13-L146

```solidity
file:   StaderOracle.sol

147      if (
            submissionCount == trustedNodesCount / 2 + 1 &&
            _exchangeRate.reportingBlockNumber > exchangeRate.reportingBlockNumber
        )
186      if (
            !staderConfig.onlyManagerRole(msg.sender) &&
            erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY > block.number
        )  
317      if (
            submissionCount == trustedNodesCount / 2 + 1 &&
            _validatorStats.reportingBlockNumber > validatorStats.reportingBlockNumber
        )      
431       if (
            submissionCount == trustedNodesCount / 2 + 1 &&
            _withdrawnValidators.reportingBlockNumber > reportingBlockNumberForWithdrawnValidators
        )
664          if (
            !(newExchangeRate >= (currentExchangeRate * (ER_CHANGE_MAX_BPS - erChangeLimit)) / ER_CHANGE_MAX_BPS &&
                newExchangeRate <= ((currentExchangeRate * (ER_CHANGE_MAX_BPS + erChangeLimit)) / ER_CHANGE_MAX_BPS))
        )


```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL147C7-L150
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL186C8-L189
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL371C8-L374
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL431C7-L434
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL664C4-L667


```solidity
file:   ValidatorWithdrawalVault.sol
35      if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold())

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#LL35C9-L35

### [G-06]  Do not calculate constants to save gas:

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

In Solidity, a constant is a value that is known at compile-time and cannot be changed during runtime. Constants can be defined using the constant keyword or the immutable keyword.
The statement "Do not calculate constants to save gas" means that if a constant value can be calculated at compile-time, it is more gas-efficient to define the constant directly with the calculated value rather than calculating it at runtime. This is because calculating a constant at runtime requires additional gas to be consumed, whereas defining it directly with the calculated value does not.

For example, consider the following code:

```solidity
contract MyContract {
  uint256 constant public FOO = 10 * 5;
  
  function useFoo() public pure returns (uint256) {
    return FOO;
  }
}
```
In this code, the constant FOO is defined as the product of 10 and 5, which is known at compile-time. By defining the constant directly with the calculated value, the gas cost of the contract is reduced, as the multiplication operation does not need to be performed at runtime.

On the other hand, if the constant value cannot be calculated at compile-time, it may be more gas-efficient to calculate it at runtime rather than defining it as a constant. This is because defining a constant with a complex or computationally-expensive value can increase the gas cost of the contract, while calculating the value at runtime may only consume gas when the function is actually called.

Therefore, when defining constants in a smart contract, it is important to consider whether the value can be calculated at compile-time and whether defining it as a constant would result in unnecessary gas consumption.

```solidity
file:  StaderOracle.sol
26     uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL26C5-L26


### [G-07]  Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
In Solidity, the address(this) expression returns the address of the current contract instance. This expression is commonly used to send or receive Ether to or from the contract.

The statement "Use hardcoded address instead of address(this) to save gas cost" means that using a hardcoded address instead of the address(this) expression can be more gas-efficient when sending or receiving Ether to or from the contract. This is because using the address(this) expression requires additional gas to be consumed to retrieve the address of the current contract, while using a hardcoded address does not.

For example, consider the following code:
```solidity
contract MyContract {
  address public myAddress = 0x1234567890123456789012345678901234567890;
  
  function receiveEther() public payable {
    // Receive Ether using address(this)
    // msg.value is the value of the Ether being sent
    address(this).transfer(msg.value);
  }
  
  function receiveEtherHardcoded() public payable {
    // Receive Ether using hardcoded address
    // msg.value is the value of the Ether being sent
    myAddress.transfer(msg.value);
  }
}
```
In this code, the receiveEther() function receives Ether using address(this), while the receiveEtherHardcoded() function receives Ether using a hardcoded address. Although both functions achieve the same result, the receiveEtherHardcoded() function is more gas-efficient, as it does not require additional gas to retrieve the address of the current contract.

Therefore, using a hardcoded address instead of the address(this) expression can be a gas-efficient way to send or receive Ether to or from a smart contract, as it reduces the gas cost of the contract. However, it is important to ensure that the hardcoded address is correct and does not change during the contract's lifetime.


```solidity
file:    Auction.sol
55       if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#LL55C8-L55


```solidity
file:   StaderInsuranceFund.sol
43      if (address(this).balance < _amount || _amount == 0
62      if (address(this).balance < _amount)
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#LL43C9-L43
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#LL62C9-L62

```solidity
file:   NodeELRewardVault.sol

25      uint8 poolId = IVaultProxy(address(this)).poolId();
        uint256 operatorId = IVaultProxy(address(this)).id();
        IStaderConfig staderConfig = IVaultProxy(address(this)).staderConfig();
        uint256 totalRewards = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#LL25C1-L28

```solidity
file:   PermissionedPool.sol
174     if (preDepositValidatorCount != 0 || address(this).balance == 0) 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#LL174C9-L174

```solidity
file:   SDCollateral.sol
47      if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount))
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#LL47C9-L47

```solidity
file:    SocializingPool.sol
67             if (
            _rewardsData.operatorETHRewards + _rewardsData.userETHRewards + _rewardsData.protocolETHRewards >
            address(this).balance - totalOperatorETHRewardsRemaining
        ) {
            revert InsufficientETHRewards();
        }
        if (
            _rewardsData.operatorSDRewards >
            IERC20(staderConfig.getStaderToken()).balanceOf(address(this)) - totalOperatorSDRewardsRemaining
        ) 

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#LL67C2-L76


## [G-08] Use calldata instead of memory for function arguments that do not get mutated

In Solidity, function arguments can be passed as either calldata or memory types. The calldata type is a read-only area where the function arguments are stored, and it is cheaper to access than memory. The memory type is a read-write area where variables are stored, and it can be more expensive to access than calldata.
If a function argument does not get mutated within the function, it is more efficient to pass it as a calldata argument instead of a memory argument. This is because calldata is cheaper to access than memory, and it avoids the overhead of copying the data from calldata to memory


```solidity
file:     PermissionedNodeRegistry.sol
    
192         function allocateValidatorsAndUpdateOperatorId(uint256 _numValidators)
        external
        override
        returns (uint256[] memory selectedOperatorCapacity)
    {
        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());
        // nextOperatorId is total operator count plus 1
        selectedOperatorCapacity = new uint256[](nextOperatorId);

        uint256 validatorPerOperator = _numValidators / totalActiveOperatorCount;
        uint256[] memory remainingOperatorCapacity = new uint256[](nextOperatorId);
        uint256 totalValidatorToDeposit;
        bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);
        if (validatorPerOperatorGreaterThanZero) {
            for (uint256 i = 1; i < nextOperatorId; ) {
                if (!operatorStructById[i].active) {
                    continue;
                }
                remainingOperatorCapacity[i] = getOperatorQueuedValidatorCount(i);
                selectedOperatorCapacity[i] = Math.min(remainingOperatorCapacity[i], validatorPerOperator);
                totalValidatorToDeposit += selectedOperatorCapacity[i];
                remainingOperatorCapacity[i] -= selectedOperatorCapacity[i];
                unchecked {
                    ++i;
                }
            }
        }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#LL192C1-L2183

## Recommanded code: 

```solidity

  function allocateValidatorsAndUpdateOperatorId(uint256 _numValidators)
        external
        override
    -    returns (uint256[] memory selectedOperatorCapacity)
    +    returns (uint256[] calldata selectedOperatorCapacity)

    {
        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());
        // nextOperatorId is total operator count plus 1
        selectedOperatorCapacity = new uint256[](nextOperatorId);

        uint256 validatorPerOperator = _numValidators / totalActiveOperatorCount;
    -    uint256[] memory remainingOperatorCapacity = new uint256[](nextOperatorId);
    +    uint256[] calldata remainingOperatorCapacity = new uint256[](nextOperatorId);
        uint256 totalValidatorToDeposit;
        bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);
        if (validatorPerOperatorGreaterThanZero) {
            for (uint256 i = 1; i < nextOperatorId; ) {
                if (!operatorStructById[i].active) {
                    continue;
                }
                remainingOperatorCapacity[i] = getOperatorQueuedValidatorCount(i);
                selectedOperatorCapacity[i] = Math.min(remainingOperatorCapacity[i], validatorPerOperator);
                totalValidatorToDeposit += selectedOperatorCapacity[i];
                remainingOperatorCapacity[i] -= selectedOperatorCapacity[i];
                unchecked {
                    ++i;
                }
            }
        }


```

```solidity
file:  PermissionedPool.sol
95     uint256[] memory selectedOperatorCapacity = IPermissionedNodeRegistry(nodeRegistryAddress)
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#LL95C9-L95


## [G-09] Gas overflow during iteration (DoS)
The issue of gas overflow during iteration (DoS) is important to be aware of when designing and implementing loops in smart contracts to prevent potential DoS attacks.
If a loop requires a large amount of gas per iteration, an attacker could intentionally create a transaction that calls the function with a small amount of gas, causing the loop to fail and the transaction to revert. This could prevent other transactions from being processed and lead to a denial-of-service (DoS) attack.
Therefore, it's important to design loops that have a reasonable gas cost per iteration, and to test the gas usage of loops under various conditions to ensure that they don't exceed the gas limit for a block. If a loop requires a large amount of gas per iteration, it may be necessary to break the loop into smaller pieces or to use alternative algorithms that are more gas-efficient.


```solidity
file:

98       function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
        uint256 reportedValidatorCount = _pubkey.length;
        for (uint256 i; i < reportedValidatorCount; ) {
            if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
                revert ValidatorSettled();
            }
            bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
            // Retrieve the penalty for changing the fee recipient address based on Rated.network data.
            uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
            uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);

            // Compute the total penalty for the given validator public key,
            // taking into account additional penalties and penalty reversals from the DAO.
            uint256 totalPenalty = _mevTheftPenalty + _missedAttestationPenalty + additionalPenaltyAmount[pubkeyRoot];
            totalPenaltyAmount[_pubkey[i]] = totalPenalty;
            if (totalPenalty >= validatorExitPenaltyThreshold) {
                emit ForceExitValidator(_pubkey[i]);
            }
            unchecked {
                ++i;
            }
        }
    }

```
# Recommanded code: 

```solidity
   function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
    require(_characterList.length < max _characterListLengt, "max length");

        uint256 reportedValidatorCount = _pubkey.length;
        for (uint256 i; i < reportedValidatorCount; ) {
            if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
                revert ValidatorSettled();
            }
            bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
            // Retrieve the penalty for changing the fee recipient address based on Rated.network data.
            uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
            uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);

            // Compute the total penalty for the given validator public key,
            // taking into account additional penalties and penalty reversals from the DAO.
            uint256 totalPenalty = _mevTheftPenalty + _missedAttestationPenalty + additionalPenaltyAmount[pubkeyRoot];
            totalPenaltyAmount[_pubkey[i]] = totalPenalty;
            if (totalPenalty >= validatorExitPenaltyThreshold) {
                emit ForceExitValidator(_pubkey[i]);
            }
            unchecked {
                ++i;
            }
        }
    }

```
###  [G-10] Empty blocks should be removed or emit something

the statement "Empty blocks should be removed or emit something to save gas" is correct. Empty blocks in Solidity code can increase the gas cost of the contract, as they consume gas for the block even though they do not perform any useful operation.

If the constructor of a contract is empty, removing it can help to reduce the gas cost of the contract. This is because the constructor is executed only once during the deployment of the contract, and if it does not perform any useful operation, it can be safely removed without affecting the functionality of the contract.


```solidity
file:   VaultProxy.sol
17      constructor() {}

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L17

```solidity
file:  ValidatorWithdrawalVault.sol
23     constructor() {}
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#LL23C5-L23

```solidity
file:   NodeELRewardVault.sol
14      constructor() {}

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#LL14C5-L14


### [G-11] public functions not called by the contract should be declared external instead

This is because external functions have less overhead than public functions.
When a public function is called, the Solidity compiler creates an internal function to handle the call, as well as additional code to check the function's access level and arguments. This additional overhead can increase the gas cost of calling the function. In contrast, external functions do not have this overhead, as they can only be called from outside the contract and do not need to perform the same access level and argument checks.
Therefore, if a public function is not called from within the same contract, declaring it as external instead of public can reduce the gas cost of the contract

```solidity
file:   ETHx.sol
29      function initialize(address _admin, address _staderConfig) public initializer 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#LL29C5-L29

```solidity
file:  PermissionedNodeRegistry.sol

66    function initialize(address _admin, address _staderConfig) public initializer

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#LL66C5-L66

```solidity
file:   PermissionedPool.sol
40      function initialize(address _admin, address _staderConfig) public initializer 

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#LL40C5-L40

```solidity
file:   PermissionlessNodeRegistry.sol
440     function getTotalQueuedValidatorCount() public view override returns (uint256)

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#LL440C5-L440

```solidity
file:   PermissionlessPool.sol
38      function initialize(address _admin, address _staderConfig) public initializer 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#LL38C5-L38

```solidity
file:   PoolUtils.sol
25      function initialize(address _admin, address _staderConfig) public initializer
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#LL25C5-L25

```solidity
file:   SDCollateral.sol
26      function initialize(address _admin, address _staderConfig) public initializer 
205     function convertSDToETH(uint256 _sdAmount) public view override returns (uint256)
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#LL26C4-L26
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#LL205C4-L205

```solidity
file:   SocializingPool.sol
39      function initialize(address _admin, address _staderConfig) public initializer 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#LL39C4-L39



### [G-12]  Use != 0 instead of > 0 for unsigned integer comparison to save gas 

The statement "Use != 0 instead of > 0 for unsigned integer comparison to save gas cost" means that using the != 0 comparison operator is more gas-efficient than using the > 0 comparison operator when comparing unsigned integers. This is because the Solidity compiler can optimize the != 0 comparison to a single operation, while the > 0 comparison requires two operations.

For example, consider the following code:

```solidity
function compareUnsignedIntegers(uint256 a) public pure returns (bool) {
  if (a != 0) {
    return true;
  } else {
    return false;
  }
}

function compareUnsignedIntegersGreaterThanZero(uint256 a) public pure returns (bool) {
  if (a > 0) {
    return true;
  } else {
    return false;
  }
}

```
In the first function, compareUnsignedIntegers(), the != 0 operator is used to compare the unsigned integer a to zero. In the second function, compareUnsignedIntegersGreaterThanZero(), the > 0 operator is used instead.

While both functions return the same result, the compareUnsignedIntegers() function is more gas-efficient than the compareUnsignedIntegersGreaterThanZero() function. This is because the != 0 operator can be optimized to a single operation, while the > 0 operator requires two operations (one to compare to zero and another to check if the result is greater than zero).

In general, using the != 0 operator for unsigned integer comparisons can result in slightly lower gas costs than using the > 0 operator, which can be beneficial for optimizing the gas consumption of a smart contract.

```solidity
file:    PermissionedNodeRegistry.sol
299     if (totalDefectedKeys > 0) 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#LL299C9-L299


```solidity
file:  PermissionlessNodeRegistry.sol
209    if (frontRunValidatorsLength > 0)
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#LL209C9-L209

```solidity
file:     SocializingPool.sol
119      if (totalAmountETH > 0)
127      if (totalAmountSD > 0)
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#LL119C8-L119
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#LL127C9-L127

```solidity
file:    ValidatorWithdrawalVault.sol
115      if (totalRewards > 0) 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#LL115C8-L115

# Recommanded code: 

```solidity
if (totalRewards != 0) 

```




