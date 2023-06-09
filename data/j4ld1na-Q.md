|     | Issues                                                                         | Instances |
| :-: | :----------------------------------------------------------------------------- | --------: |
| 01  | Numeric values having to do with time should use `time units` for readability. |         1 |
| 02  | Default constructor.                                                           |         2 |

## 1. Numeric values having to do with time should use `time units` for readability.

_There are [units](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#time-units) for seconds, minutes, hours, days, and weeks, and since they’re defined, they should be used._

Therre an instance:

[Auction.sol#L22](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L22)

```java
 uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours
```

Recommended Mitigation Steps:

-   Use time-units.

## 2. Default constructor.

_If there is no constructor, the contract will assume the default constructor, which is equivalent to `constructor() {}`. [For example](https://docs.soliditylang.org/en/v0.8.13/contracts.html#constructor)_

There are 2 instances:

[contracts/NodeELRewardVault.sol#L14](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L14)

```ts
constructor() {}
```

[contracts/ValidatorWithdrawalVault.sol#L23](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L23)

```ts
constructor() {}
```

[contracts/VaultProxy.sol#L17](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L17)

```ts
constructor() {}
```

Recommended Mitigation Steps:

-   Delete constructor empty
