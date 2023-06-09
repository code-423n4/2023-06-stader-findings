|      | Issues                                                   | Instances |
| :--- | :------------------------------------------------------- | --------: |
| G-01 | Use assembly to check for `address(0)`.                  |         1 |
| G-02 | Use `assembly` to write storage values. Save gas.        |        29 |
| G-03 | Use `assembly` when getting a contract’s balance of ETH. |         3 |

## 1. Use assembly to check for address(0).

Example:

```java
function ownerNotZero(address _addr) public pure {
    require(_addr != address(0), "zero address)");
}

Gas: 258
```

```java
function assemblyOwnerNotZero(address _addr) public pure {
    assembly {
        if iszero(_addr) {
            mstore(0x00, "zero address")
            revert(0x00, 0x20)
        }
    }
}

Gas: 252
```

There an instance:

[contracts/Auction.sol#L21-L23](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol)

```java
    function checkNonZeroAddress(address _address) internal pure {
        if (_address == address(0)) revert ZeroAddress();
    }
```

Recommended Mitigation Steps:

-   See the example

## 2. Use `assembly` to write storage values. Save gas.

Example:

```java
address owner = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84;

function updateOwner(address newOwner) public {
    owner = newOwner;
}

Gas: 5302
```

```java
address owner = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84;

function assemblyUpdateOwner(address newOwner) public {
    assembly {
        sstore(owner.slot, newOwner)
    }
}

Gas: 5236
```

There are 29 instances:

[Auction.sol#L38-L40](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L38-L40)

```ts
duration = 2 * MIN_AUCTION_DURATION;
bidIncrement = 5e15;
nextLot = 1;
```

[contracts/PermissionedNodeRegistry.sol#L73-L79](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L73-L79)

```ts
nextOperatorId = 1;
nextValidatorId = 1;
operatorIdForExcessDeposit = 1;
inputKeyCountLimit = 50;
maxOperatorId = 10;
maxNonTerminalKeyPerOperator = 50;
verifiedKeyBatchSize = 50;
```

[contracts/PermissionedPool.sol#L45-L46](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L45-L46)

```ts
protocolFee = 500;
operatorFee = 500;
```

[contracts/PermissionlessNodeRegistry.sol#L73-L77](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L73-L77)

```ts
nextOperatorId = 1;
nextValidatorId = 1;
inputKeyCountLimit = 30;
maxNonTerminalKeyPerOperator = 50;
verifiedKeyBatchSize = 50;
```

[contracts/PermissionlessPool.sol#L43-L44](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L43-L44)

```ts
protocolFee = 500;
operatorFee = 500;
```

[contracts/PoolSelector.sol#L40](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L40)

```ts
poolAllocationMaxSize = 50;
```

[contracts/StaderStakePoolsManager.sol#L57](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L57)

```ts
excessETHDepositCoolDown = 3 * 7200;
```

[contracts/UserWithdrawalManager.sol#L61-L64](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L61-L64)

```ts
nextRequestIdToFinalize = 1;
nextRequestId = 1;
finalizationBatchLimit = 50;
maxNonRedeemedUserRequestCount = 1000;
```

[contracts/VaultProxy.sol#L30-L33](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L30-L33)

```ts
isValidatorWithdrawalVault = _isValidatorWithdrawalVault;
isInitialized = true;
poolId = _poolId;
id = _id;
```

## 3. Use `assembly` when getting a contract’s balance of ETH.

_You can use `selfbalance()` instead of `address(this).balance` when getting your contract’s balance of ETH to save gas._

Example:

```java
function addressInternalBalance() public returns (uint256) {
    return address(this).balance;
}

Gas: 148
```

```java
function assemblyInternalBalance() public returns (uint256) {
    assembly {
        let c := selfbalance()
        mstore(0x00, c)
        return(0x00, 0x20)
    }
}

Gas: 133
```

There are 3 instances:

[NodeELRewardVault.sol#L28](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L28)

```java
uint256 totalRewards = address(this).balance;
```

[contracts/ValidatorWithdrawalVault.sol#L34](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L34)

```java
uint256 totalRewards = address(this).balance;
```

[contracts/ValidatorWithdrawalVault.sol#L99](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L99)

```java
uint256 contractBalance = address(this).balance;
```
