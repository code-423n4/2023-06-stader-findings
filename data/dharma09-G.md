## [G-01] Use cached value for emit function

see `@audit` tag

*There is 2 instance of this issue:*

- https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol

```jsx
File: [/contracts/Auction.sol](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol)
	148: emit AuctionDurationUpdated(duration); //@audit use cache value
```

- https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L536

```solidity
File: contracts/StaderOracle.sol
	536: emit UpdatedERChangeLimit(erChangeLimit);
```

## Mitigation

```diff
- 148: emit AuctionDurationUpdated(duration);
+ 148: emit AuctionDurationUpdated(_duration);
```

## [G-02] ****Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)****

There are 4 instances of this issuse.

- [https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#LL44](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#LL68C87-L68C87)

```solidity
File: [/contracts/SDCollateral.sol](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#LL68C87-L68C87)
	44: address operator = msg.sender;
	59: address operator = msg.sender;
```

- https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L47

```solidity
File: /[contracts/OperatorRewardsCollector.sol](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L47)
	47: address operator = msg.sender;
```

- https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#LL113C8-L113C39

```solidity
File: [/contracts/SocializingPool.sol](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#LL113C8-L113C39)
	113: address operator = msg.sender;
```

## [G‑03] Using `storage` instead of `memory` for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct

*There is one instance of this issue:*

- [https://github.com/code-423n4/2023-06-stader/blob/d5f7854fdf70547c6476c00be5d97c85f2c8d064/contracts/UserWithdrawalManager.sol#L169](https://github.com/code-423n4/2023-06-stader/blob/d5f7854fdf70547c6476c00be5d97c85f2c8d064/contracts/UserWithdrawalManager.sol#L133)

```solidity
File: contracts/UserWithdrawalManager.sol
	169: UserWithdrawInfo memory userRequest = userWithdrawRequests[_requestId];
```