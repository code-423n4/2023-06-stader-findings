
## Summary 

### Gas Optimization

|no |Issue|Instances|
|----|-------|------|
| [G-01] | Can Make The Variable Outside The Loop To Save Gas | 12 | - | 
| [G-02] | Amounts should be checked for 0 before calling a transfer | 4 | - | 
| [G-03] | Change public state variable visibility to private | 1 | - | 
| [G-04] | use nestead if instead of && to save gas | 6 | - | 
| [G-05] | Do not calculate constants to save gas: | 1 | - | 
| [G-06] | Use hardcode address instead address(this) | 7 | - | 
| [G-07] | Gas overflow during iteration (DoS) | 1 | - | 
| [G-08] | Empty blocks should be removed or emit something | 3 | - | 
| [G-09] | Use != 0 instead of > 0 for unsigned integer comparison to save gas | 5 | - | 
| [G-10] | Use constants instead of type(uintx).max | 1 | - | 
| [G-11] | Unnecessary look up in if condition | 4 | - | 
| [G-12] | abi.encode() is less efficient than abi.encodePacked() to save gas. | 5 | - | 
| [G-13] | Use assembly to write address storage values | 22 | - | 
| [G-14] | Sort Solidity operations using short-circuit mode | 13 | - | 
| [G-15] | Use != 0 instead of > 0 for unsigned integer comparison | 4 | - | 
| [G-16] | Use Assembly To Check For address(0) | 1 | - |
| [G-17] | Duplicated require()/if() checks should be refactored to a modifier or function | 3 | - | 
| [G-18] | Use assembly to hash instead of Solidity | 13 | - |
| [G-19] | use Mappings Instead of Arrays. | 2 | - |
| [G-20] | Use assembly for math (add, sub, mul, div)| 2 | - |
| [G-21] | Use assembly when getting a contract’s balance of ETH | 4 | - |
| [G-22] | Gas savings can be achieved by changing the model for assigning value to the structure (260 gas)  | 1 | - |
| [G-22] | Change Constant to Immutable for keccak Variables (20 gas per keccak)  | 49 | - |



## [G-01]  Can Make The Variable Outside The Loop To Save Gas 

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






## [G-02]  Amounts should be checked for 0 before calling a transfer

In Solidity, the transfer() function is commonly used to transfer ether or tokens from one address to another. When calling the transfer() function, it is important to check the amount being transferred to ensure that it is greater than 0, as transferring 0 amounts is not necessary and can consume unnecessary gas.

The statement "Amounts should be checked for 0 before calling a transfer to save gas" means that checking the amount being transferred before calling the transfer() function can help to reduce the gas cost of the contract and optimize the Solidity code.



```solidity
file:  Auction.sol
55    if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
87    if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L55
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L87

```solidity
82    super._beforeTokenTransfer(from, to, amount);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L82

```solidity
68    if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L68



## [G-03]  Change public state variable visibility to private

it's generally a good practice to limit the visibility of state variables to the minimum necessary level. This means that you should use the private visibility modifier for state variables whenever possible, and avoid using the public modifier unless it is absolutely necessary.

1.Security: Public state variables can be read and modified by anyone on the blockchain, which can make your contract vulnerable to attacks. By using the private modifier, you can limit access to your state variables and reduce the risk of malicious actors exploiting them.

2.Encapsulation: Using private state variables can help to encapsulate the internal workings of your contract and make it easier to reason about and maintain. By only exposing the necessary interfaces to the outside world, you can reduce the complexity and potential for errors in your contract.

3.Gas costs: Public state variables can be more expensive to read and write than private state variables, since Solidity generates additional getter and setter functions for public variables. By using private state variables, you can reduce the gas cost of your contract and improve its efficiency.

```solidity
File: /contracts/VaultProxy.sol
14   address public override owner;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L14




## [G-04]  use nestead if instead of && to save gas

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

## [G-05]  Do not calculate constants to save gas:

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


## [G-06]  Use hardcode address instead address(this)

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



## [G-07]  Gas overflow during iteration (DoS)
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
### Recommanded code: 

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


## [G-08]  Empty blocks should be removed or emit something

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


## [G-09]  Use != 0 instead of > 0 for unsigned integer comparison to save gas 

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

### Recommanded code: 

```solidity
if (totalRewards != 0) 

```


## [G-10]  Use constants instead of type(uintx).max

In Solidity, type(uintX).max is used to represent the maximum value for an unsigned integer of X bits. This value is often used in contracts to represent an "infinity" value or indicate that a certain condition has not been met.
However, using type(uintX).max can be expensive in terms of gas cost since it requires the compiler to perform a complex computation to determine the maximum value for an unsigned integer of X bits.
To save gas, it's recommended to use constants instead of type(uintX).max to represent infinity or other large values. For example, we can define a constant MAX_UINT256 that represents the maximum value for an unsigned 256-bit integer: 

```solidity
// Inefficient code that uses type(uint256).max
function doSomething(uint256 value) public {
    require(value != type(uint256).max, "Value must not be infinity");
    // ...
}

// Efficient code that uses a constant
contract MyContract {
    uint256 constant MAX_UINT256 = 2**256 - 1;

    function doSomething(uint256 value) public {
        require(value != MAX_UINT256, "Value must not be infinity");
        // ...
    }
}

```
In this optimized version, we define a constant MAX_UINT256 that represents the maximum value for an unsigned 256-bit integer. We then use this constant in the doSomething() function to check whether the input value is equal to infinity.

Using constants instead of type(uintX).max can improve gas efficiency by avoiding the need for the compiler to perform a complex computation to determine the maximum value for an unsigned integer of X bits.

```solidity
file:    SDCollateral.sol
106      IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#LL106C8-L106



## [G-11]  Unnecessary look up in if condition

If the || condition isn’t required, the second condition will have been looked up unnecessarily.
In Solidity, if statements are commonly used to check conditions and execute code based on the result. When using an if statement with multiple conditions joined by the logical OR operator (||), it is important to ensure that the conditions are evaluated in the correct order to avoid unnecessary lookups.

The statement "Unnecessary lookup in if condition: If the || condition isn’t required, the second condition will have been looked up unnecessarily" means that if the first condition in an if statement joined by || is true, the second condition will not be evaluated, but if the first condition is false, the second condition will be evaluated. Therefore, if the second condition is not required, it will have been looked up unnecessarily, consuming additional gas.

For example, consider the following code:


```solidit
contract MyContract {
  uint256 public myNumber;
  
  function setNumber(uint256 _number) public {
    if (_number > myNumber || _number == 0) {
      myNumber = _number;
    }
  }
}
```
In this code, the setNumber() function checks if _number is greater than myNumber or if _number is equal to zero before setting myNumber. However, if _number is greater than myNumber, the second condition (_number == 0) is not required and will have been looked up unnecessarily, consuming additional gas.

Therefore, when using an if statement with multiple conditions joined by ||, it is important to ensure that the conditions are evaluated in the correct order to avoid unnecessary lookups and reduce the gas cost of the contract. In this case, the condition that is most likely to be true should be evaluated first, to minimize the number of unnecessary lookups.


```solidity
file:    PermissionedPool.sol
174     if (preDepositValidatorCount != 0 || address(this).balance == 0)

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#LL174C9-L174

```solidity
file:      PoolUtils.sol
282       if (
            INodeRegistry(IStaderPoolBase(_poolAddress).getNodeRegistry()).POOL_ID() != _poolId ||
            isExistingPoolId(_poolId)
        )

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#LL282C7-L285

```solidity
file:   SDCollateral.sol
127     if ((_minThreshold > _withdrawThreshold) || (_minThreshold > _maxThreshold)) 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#LL127C9-L127

```solidity
file:  SocializingPool.sol
170   if (_index == 0 || _index > lastReportedRewardsData.index)

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#LL170C9-L170




## [G-12]  abi.encode() is less efficient than abi.encodePacked() to save gas.
In Solidity, abi.encode() and abi.encodePacked() are used to encode function arguments and data into a byte array for storage or transmission. However, abi.encodePacked() is generally more gas-efficient than abi.encode() because it does not add padding to the encoded data.

When using abi.encode(), the function arguments are encoded with padding to ensure that the encoded data is a multiple of 32 bytes. This padding can add unnecessary zeros to the encoded data, which can increase the size and gas cost of the transaction.

On the other hand, abi.encodePacked() does not add padding to the encoded data and returns a tightly packed representation of the input data. This can reduce the size and gas cost of the transaction, as it avoids unnecessary padding.

Here is an example code snippet that demonstrates the difference in gas cost between abi.encode() and abi.encodePacked():

```solidity
contract MyContract {
    function encodeTest(uint256 _a, uint256 _b, uint256 _c) public pure returns (bytes memory) {
        return abi.encode(_a, _b, _c);
    }
    
    function encodePackedTest(uint256 _a, uint256 _b, uint256 _c) public pure returns (bytes memory) {
        return abi.encodePacked(_a, _b, _c);
    }
}
```
In this code, the encodeTest() function uses abi.encode() to encode three uint256 values, while encodePackedTest() uses abi.encodePacked() to encode the same values. When measuring the gas cost of calling these functions using a gas profiler, we can see that encodePackedTest() uses less gas than encodeTest().
Therefore, it is generally more gas-efficient to use abi.encodePacked() instead of abi.encode() when encoding data in Solidity, as it avoids unnecessary padding and reduces the size and gas cost of the transaction.

```solidity
file:       StaderOracle.sol
126              bytes32 nodeSubmissionKey = keccak256(
            abi.encode(
                msg.sender,
                _exchangeRate.reportingBlockNumber,
                _exchangeRate.totalETHBalance,
                _exchangeRate.totalETHXSupply
            )
        );
        bytes32 submissionCountKey = keccak256(
            abi.encode(_exchangeRate.reportingBlockNumber, _exchangeRate.totalETHBalance, _exchangeRate.totalETHXSupply)
        );

220             bytes32 nodeSubmissionKey = keccak256(
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
        bytes32 submissionCountKey = keccak256(
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
333             bytes32 nodeSubmissionKey = keccak256(
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
        bytes32 submissionCountKey = keccak256(
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
407     bytes memory encodedPubkeys = abi.encode(_withdrawnValidators.sortedPubkeys);
        // Get submission keys
        bytes32 nodeSubmissionKey = keccak256(
            abi.encode(
                msg.sender,
                _withdrawnValidators.reportingBlockNumber,
                _withdrawnValidators.nodeRegistry,
                encodedPubkeys
            )
        );
        bytes32 submissionCountKey = keccak256(
            abi.encode(_withdrawnValidators.reportingBlockNumber, _withdrawnValidators.nodeRegistry, encodedPubkeys)
        );
466     bytes memory encodedPubkeys = abi.encode(_mapd.sortedPubkeys);

        // Get submission keys
        bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _mapd.index, encodedPubkeys));
        bytes32 submissionCountKey = keccak256(abi.encode(_mapd.index, encodedPubkeys));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL126C1-L136

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL220C1-L242

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL333C1-L355

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL407C1-L419C11

### Recommanded code: 

```solidity
        bytes32 nodeSubmissionKey = keccak256(
            abi.encodePacked(
                msg.sender,
                _exchangeRate.reportingBlockNumber,
                _exchangeRate.totalETHBalance,
                _exchangeRate.totalETHXSupply
            )
        );
        bytes32 submissionCountKey = keccak256(
            abi.encodePacked(_exchangeRate.reportingBlockNumber, _exchangeRate.totalETHBalance, _exchangeRate.totalETHXSupply)
        );
        bytes32 nodeSubmissionKey = keccak256(
            abi.encodePacked(
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
        bytes32 submissionCountKey = keccak256(
            abi.encodePacked(
                _rewardsData.index,
                _rewardsData.merkleRoot,
                _rewardsData.poolId,
                _rewardsData.operatorETHRewards,
                _rewardsData.userETHRewards,
                _rewardsData.protocolETHRewards,
                _rewardsData.operatorSDRewards
            )
        );
                bytes32 nodeSubmissionKey = keccak256(
            abi.encodePacked(
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
        bytes32 submissionCountKey = keccak256(
            abi.encodePacked(
                _validatorStats.reportingBlockNumber,
                _validatorStats.exitingValidatorsBalance,
                _validatorStats.exitedValidatorsBalance,
                _validatorStats.slashedValidatorsBalance,
                _validatorStats.exitingValidatorsCount,
                _validatorStats.exitedValidatorsCount,
                _validatorStats.slashedValidatorsCount
            )
        );
         bytes memory encodedPubkeys = abi.encodePacked(_withdrawnValidators.sortedPubkeys);
        // Get submission keys
        bytes32 nodeSubmissionKey = keccak256(
            abi.encodePacked(
                msg.sender,
                _withdrawnValidators.reportingBlockNumber,
                _withdrawnValidators.nodeRegistry,
                encodedPubkeys
            )
        );
        bytes32 submissionCountKey = keccak256(
            abi.encodePacked(_withdrawnValidators.reportingBlockNumber, _withdrawnValidators.nodeRegistry, encodedPubkeys)
        );
        bytes memory encodedPubkeys = abi.encodePacked(_mapd.sortedPubkeys);

        // Get submission keys
        bytes32 nodeSubmissionKey = keccak256(abi.encodePacked(msg.sender, _mapd.index, encodedPubkeys));
        bytes32 submissionCountKey = keccak256(abi.encodePacked(_mapd.index, encodedPubkeys));

```




##  [G-13] Use assembly to write address storage values
By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```solidity
File: Auction.sol
37    staderConfig = IStaderConfig(_staderConfig);

140    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L37
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L140

```solidity
File: ETHx.sol
37    staderConfig = IStaderConfig(_staderConfig);

87    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L37
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L87

```solidity
File: /contracts/OperatorRewardsCollector.sol
34    staderConfig = IStaderConfig(_staderConfig);

41    balances[_receiver] += msg.value;

58    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L34
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L41
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L58

```solidity
File:  Penalty.sol
42    staderConfig = IStaderConfig(_staderConfig);

43    ratedOracleAddress = _ratedOracleAddress;

86    ratedOracleAddress = _ratedOracleAddress;

93    staderConfig = IStaderConfig(_staderConfig);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L42
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L43
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L86
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L




## [G-014] Sort Solidity operations using short-circuit mode

Short-circuiting is a technique used in Solidity contract development that utilizes logical operators (OR/AND) to control the order of execution of different operations with varying gas costs. It involves placing the operations with lower gas costs at the beginning and those with higher gas costs at the end. This way, if the low-cost operation at the beginning is successful, the subsequent high-cost Ethereum virtual machine operation can be skipped, resulting in significant gas savings. Essentially, short-circuiting allows the program to take a shortcut and bypass more expensive operations when it's possible to do so.

example:
function transfer(address recipient, uint amount) public {
    require(recipient != address(0) || amount > 0, "Invalid transfer parameters");
    // Perform transfer logic
}



```solidity
file:  PermissionedNodeRegistry.sol
693   if (_pubkeyLength != _preDepositSignatureLength || _pubkeyLength != _depositSignatureLength) {
697   if (keyCount == 0 || keyCount > inputKeyCountLimit) {
174   if (preDepositValidatorCount != 0 || address(this).balance == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L693

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L697

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L174

```solidity
file:  PermissionlessNodeRegistry.sol
649   if (_pubkeyLength != _preDepositSignatureLength || _pubkeyLength != _depositSignatureLength) {
653   if (keyCount == 0 || keyCount > inputKeyCountLimit) {
713   if (_validatorId == 0 || validatorRegistry[_validatorId].status != ValidatorStatus.INITIALIZED) { 
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L649

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L653

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L713

```solidity
file:   PoolSelector.sol
103     if (ethToDeposit < ETH_PER_NODE || selectedValidatorCount >= poolAllocationMaxSize) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L103

```solidity
file:  SDCollateral.sol
127   if ((_minThreshold > _withdrawThreshold) || (_minThreshold > _maxThreshold)) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L127

```solidity
file:   SocializingPool.sol
170   if (_index == 0 || _index > lastReportedRewardsData.index) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L170

```solidity
file:  StaderInsuranceFund.sol
44   if (address(this).balance < _amount || _amount == 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L44

```solidity
file:   StaderOracle.sol
532   if (_erChangeLimit == 0 || _erChangeLimit > ER_CHANGE_MAX_BPS) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L532

```solidity
file: StaderStakePoolsManager.sol
171   if (assets > maxDeposit() || assets < minDeposit()) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L171

```solidity
file:  UserWithdrawalManager.sol
98    if (assets < staderConfig.getMinWithdrawAmount() || assets > staderConfig.getMaxWithdrawAmount()) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L98

## [G-15]  Use != 0 instead of > 0 for unsigned integer comparison

When we say "use != 0 instead of > 0 for unsigned integer comparison," we mean that when checking if an unsigned integer value is non-zero, it's more gas-efficient to use the != operator instead of the > operator. This is because the != operator is a bitwise operation and is therefore more efficient than the arithmetic comparison performed by the > operator.

Here's an example of how to use != 0 instead of > 0 for unsigned integer comparison:

contract MyContract {
    uint public value;
    
    function isNonZero() public view returns (bool) {
        return value != 0;
    }
}


```solidity
file:   PermissionedNodeRegistry.sol
299   if (totalDefectedKeys > 0) {
209   if (frontRunValidatorsLength > 0) {  
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L299
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L209


```solidity
119   if (totalAmountETH > 0) {
127   if (totalAmountSD > 0) {
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L119
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L127

## [G-16]  Use Assembly To Check For address(0)

```solidity
file:  UserWithdrawalManager.sol
96    if (_owner == address(0)) revert ZeroAddressReceived();
```
 https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L96

## [G-17]  Duplicated require()/if() checks should be refactored to a modifier or function


```solidity
file:  UtilLib.sol
59    if (_addr != withdrawVaultAddress) {
75    if (_addr != withdrawVaultAddress) {
90    if (_addr != withdrawVaultAddress) {   
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L59

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L75

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L90

 ## [G-18] Use assembly to hash instead of Solidity

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
file:  SocializingPool.sol
174    bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L174

```solidity
file:  StaderOracle.sol
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
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L126

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L134

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L220

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L232

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L282

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L283

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L333

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L345

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L409

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L417

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L469

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L470


```solidity
file:    VaultFactory.sol
16       bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L16


## [G-19]  use Mappings Instead of Arrays.



When we say "use mappings instead of arrays," we mean that when storing and accessing data in a smart contract, it's more gas-efficient to use mappings instead of arrays. This is because mappings do not require iteration to find a specific element, whereas arrays do.


```solidity
file: Penalty.sol
125   uint256[] memory violatedEpochs = IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L125

```solidity
file:   PermissionedNodeRegistry.sol
202    uint256[] memory remainingOperatorCapacity = new uint256[](nextOperatorId);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L202



## [G-20]| Use assembly for math (add, sub, mul, div)

When we say "use assembly for math operations," we mean that when performing basic math operations in a smart contract, it's more gas-efficient to use assembly code to perform the operation instead of Solidity's built-in math functions. This is because Solidity's built-in math functions require additional gas consumption to perform the operation.



```solidity
file: PermissionedNodeRegistry.sol
500   uint256 endIndex = startIndex + _pageSize;
629  uint256 endIndex = startIndex + _pageSize;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L500
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L629


## [G-21]  Use assembly when getting a contract’s balance of ETH

When we say "use assembly to get a contract's balance of Ether," we mean that when retrieving the balance of Ether held by a contract, it's more gas-efficient to use assembly code to perform the operation instead of Solidity's built-in address.balance property. This is because Solidity's built-in address.balance property requires additional gas consumption to perform the operation.

```solidity
file:  NodeELRewardVault.sol
28     uint256 totalRewards = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L28

```solidity
file:   PermissionedPool.sol
178     value: address(this).balance
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L178

```solidity
file:  ValidatorWithdrawalVault.sol
34     uint256 totalRewards = address(this).balance;
99      uint256 contractBalance = address(this).balance;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L34
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L99


## [G-22]  Gas savings can be achieved by changing the model for assigning value to the structure (260 gas) 


When we say "change the model for assigning value to a structure," we mean that when initializing or assigning values to a structure, it's more gas-efficient to use a single assignment statement that includes all of the values for the structure, instead of assigning each value one by one. This is because each individual assignment requires additional gas consumption, so reducing the number of assignments can lead to significant gas savings.

```solidity
file:   SDCollateral.sol
131      poolThresholdbyPoolId[_poolId] = PoolThresholdInfo({
            minThreshold: _minThreshold,
            maxThreshold: _maxThreshold,
            withdrawThreshold: _withdrawThreshold,
            units: _units
        });
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L131-L136


## [G-23]  Change Constant to Immutable for keccak Variables

The use of constant keccak variables results in extra hashing (and so gas). This results in the keccak operation being performed whenever the variable is used, increasing gas costs relative to just storing the output hash. Changing to immutable will only perform hashing on contract deployment which will save gas. You should use immutables until the referenced issues are implemented, then you only pay the gas costs for the computation at deploy time and you can save about 20 gas per one keccak.

```solidity
file:   ETHx.sol
21      bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');
22      bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L21
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#LL22


```solidity
file:  StaderConfig.sol

12   bytes32 public constant ETH_PER_NODE = keccak256('ETH_PER_NODE');
    //amount of ETH for pre-deposit on beacon chain i.e 1 ETH
14   bytes32 public constant PRE_DEPOSIT_SIZE = keccak256('PRE_DEPOSIT_SIZE');
    //amount of ETH for full deposit on beacon chain i.e 31 ETH
16   bytes32 public constant FULL_DEPOSIT_SIZE = keccak256('FULL_DEPOSIT_SIZE');
    // ETH to WEI ratio i.e 10**18
18    bytes32 public constant DECIMALS = keccak256('DECIMALS');
    //Total fee bips
19    bytes32 public constant TOTAL_FEE = keccak256('TOTAL_FEE');
    //maximum length of operator name string
20    bytes32 public constant OPERATOR_MAX_NAME_LENGTH = keccak256('OPERATOR_MAX_NAME_LENGTH');

22    bytes32 public constant SOCIALIZING_POOL_CYCLE_DURATION = keccak256('SOCIALIZING_POOL_CYCLE_DURATION');
23    bytes32 public constant SOCIALIZING_POOL_OPT_IN_COOLING_PERIOD =
        keccak256('SOCIALIZING_POOL_OPT_IN_COOLING_PERIOD');
24    bytes32 public constant REWARD_THRESHOLD = keccak256('REWARD_THRESHOLD');
25    bytes32 public constant MIN_DEPOSIT_AMOUNT = keccak256('MIN_DEPOSIT_AMOUNT');
26    bytes32 public constant MAX_DEPOSIT_AMOUNT = keccak256('MAX_DEPOSIT_AMOUNT');
27    bytes32 public constant MIN_WITHDRAW_AMOUNT = keccak256('MIN_WITHDRAW_AMOUNT');
28    bytes32 public constant MAX_WITHDRAW_AMOUNT = keccak256('MAX_WITHDRAW_AMOUNT');
    //minimum delay between user requesting withdraw and request finalization
30    bytes32 public constant MIN_BLOCK_DELAY_TO_FINALIZE_WITHDRAW_REQUEST =
        keccak256('MIN_BLOCK_DELAY_TO_FINALIZE_WITHDRAW_REQUEST');
31    bytes32 public constant WITHDRAWN_KEYS_BATCH_SIZE = keccak256('WITHDRAWN_KEYS_BATCH_SIZE');

33    bytes32 public constant ADMIN = keccak256('ADMIN');
34    bytes32 public constant STADER_TREASURY = keccak256('STADER_TREASURY');

36    bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');
37    bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');
38    bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');
39    bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');
40    bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');
41    bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');
42    bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');
43    bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');
44    bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');
45    bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');
46    bytes32 public constant override ETH_DEPOSIT_CONTRACT = keccak256('ETH_DEPOSIT_CONTRACT');
47    bytes32 public constant override PERMISSIONLESS_POOL = keccak256('PERMISSIONLESS_POOL');
48    bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');
49    bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');
50    bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');
51    bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');
52    bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');
53    bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');
54    bytes32 public constant override NODE_EL_REWARD_VAULT_IMPLEMENTATION =
        keccak256('NODE_EL_REWARD_VAULT_IMPLEMENTATION');
55    bytes32 public constant override VALIDATOR_WITHDRAWAL_VAULT_IMPLEMENTATION =
        keccak256('VALIDATOR_WITHDRAWAL_VAULT_IMPLEMENTATION');

    //POR Feed Proxy
58    bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');
59    bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');

    //Roles
62    bytes32 public constant override MANAGER = keccak256('MANAGER');
63    bytes32 public constant override OPERATOR = keccak256('OPERATOR');

65    bytes32 public constant SD = keccak256('SD');
66    bytes32 public constant ETHx = keccak256('ETHx');

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#LL12C1-L72


```solidity
file: StaderOracle.sol
50   bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency
51   bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key
52   bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key
53    bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key
54    bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#LL50C4-L54