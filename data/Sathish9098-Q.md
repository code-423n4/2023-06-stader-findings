# LOW FINDINGS

## [L-1] Don’t use payable.call()

payable.call() is a low-level method that can be used to send ether to a contract, but it has some limitations and risks as you've pointed out. One of the primary risks of using payable.call() is that it doesn't guarantee that the contract's payable function will be called successfully. This can lead to funds being lost or stuck in the contract

The contract does not have a payable callback
The contract’s payable callback spends more than 2300 gas (which is only enough to emit something)
The contract is called through a proxy which itself uses up the 2300 gas Use OpenZeppelin’s Address.sendValue() instead

```solidity
FILE: 2023-06-stader/contracts/Auction.sol

131:  (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#LL131C8-L131C82

```solidity
FILE: 2023-06-stader/contracts/SocializingPool.sol

91:  (bool success, ) = payable(staderConfig.getStaderTreasury()).call{value: _rewardsData.protocolETHRewards} 
     ('');
121: (success, ) = payable(operatorRewardsAddr).call{value: totalAmountETH}('');

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#LL121C13-L121C88

```solidity
FILE: 2023-06-stader/contracts/StaderStakePoolsManager.sol

102: (bool success, ) = payable(staderConfig.getUserWithdrawManager()).call{value: _amount}('');

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderStakePoolsManager.sol#L102

### Recommended Mitigation

Use OpenZeppelin’s Address.sendValue()


##

## [L-2] Use .call instead of .transfer to send ether

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues

````solidity
FILE: 2023-06-stader/contracts/Auction.sol

114:  if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#LL114C8-L114C108

```solidity
FILE: 2023-06-stader/contracts/SDCollateral.sol

68:  if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#LL68C8-L68C96

### Recommended Mitigation

Replace .transfer with .call. Note that the result of .call need to be checked. 


## [L-3] Unbounded loop

### This instance not found by bot race. This is most important Unbounded loop logic 

``nextOperatorId`` the value is only incremented .

PermissionedNodeRegistry.allocateValidatorsAndUpdateOperatorId() will iterate all the OperatorId.

Currently, ``nextOperatorId`` can grow indefinitely. E.g. there’s no maximum limit and there’s no functionality to remove OperatorId

If the value grows too large, calling PermissionedNodeRegistry.allocateValidatorsAndUpdateOperatorId() might run out of gas and revert. 

```solidity
FILE: Breadcrumbs2023-06-stader/contracts/PermissionedNodeRegistry.sol

206:  for (uint256 i = 1; i < nextOperatorId; ) {

488: for (uint256 i = 1; i < nextOperatorId; ) {

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L488

##

## [L-4] Inconsistency between the comment and the actual value assigned

 It appears that there is an inconsistency between the comment and the actual value assigned to the constant variable. The comment states that the value represents a 24-hour duration, but the assigned value of 7200 actually corresponds to a duration of 2 hours

```diff
FILE: Breadcrumbs2023-06-stader/contracts/Auction.sol

- 22: uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours
+ 22: uint256 public constant MIN_AUCTION_DURATION = 7200; // 2 hours
```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#LL22C5-L22C69

##

## [L-5] Resolve the warning to avoid unexpected behavior 

```
Warning (3628): Warning: This contract has a payable fallback function, but no receive ether function. Consider adding a receive ether function.
 --> contracts/VaultProxy.sol:8:1:
  |
8 | contract VaultProxy is IVaultProxy {
  | ^ (Relevant source part starts here and spans across multiple lines).
Note: The payable fallback function is defined here.
  --> contracts/VaultProxy.sol:41:5:
   |
41 |     fallback(bytes calldata _input) external payable returns (bytes memory) {
   |     ^ (Relevant source part starts here and spans across multiple lines).

```

##

## [L-6] Consider using upgradable ERC20 contracts 

Upgradable ERC20 contracts provide flexibility and allow for future upgrades and improvements to the contract's functionality without disrupting the existing ecosystem.

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L9

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L14

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#L15





update codes to avoid Compile Errors: 

add a timelock to critical functions:

Critical changes should use two-step procedure

Tokens accidentally sent to the contract cannot be recovered












[M‑01]	The owner is a single point of failure and a centralization risk	2
[L‑01]	Division by zero not prevented	1
[L‑02]	Initialization can be front-run	18
[L‑03]	Missing checks for address(0x0) when assigning values to address state variables	41
[L‑04]	Unsafe downcast	2
[L‑05]	Loss of precision	7
[L‑06]	Array lengths not checked	2
[L‑07]	Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions	18
[L‑08]	External calls in an un-bounded for-loop may result in a DOS	8

[N‑01]	Events are missing sender information	70
[N‑02]	Variables need not be initialized to zero	13
[N‑03]	Consider using named mappings	41
[N‑04]	Non-external/public variable and function names should begin with an underscore	78
[N‑05]	Redundant inheritance specifier	15
[N‑06]	Large numeric literals should use underscores for readability	6
[N‑07]	Unused struct definition	2
[N‑08]	Constants in comparisons should appear on the left side	55
[N‑09]	Cast to bytes or bytes32 for clearer semantic meaning	2
[N‑10]	Use bytes.concat() on bytes instead of abi.encodePacked() for clearer semantic meaning	4
[N‑11]	Custom error has no error details	120
[N‑12]	Events may be emitted out of order due to reentrancy	1
[N‑13]	Imports could be organized more systematically	2
[N‑14]	Unusual loop variable	1
[N‑15]	Adding a return statement when the function defines a named return variable, is redundant	2
[N‑16]	constants should be defined rather than using magic numbers	62
[N‑17]	Event is not properly indexed	56
[N‑18]	Vulnerable versions of packages are being used	1
[N‑19]	Import declarations should import specific identifiers, rather than the whole file	199
[N‑20]	Return values of approve() not checked	1
[N‑21]	The nonReentrant modifier should occur before all other modifiers	2
[N‑22]	Events that mark critical parameter changes should contain both the old and the new value	49
[N‑23]	Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)	3
[N‑24]	Constant redefined elsewhere	2
[N‑25]	Use @inheritdoc rather than using a non-standard annotation	7
[N‑26]	Inconsistent spacing in comments	77
[N‑27]	Lines are too long	13
[N‑28]	Variable names that consist of all capital letters should be reserved for constant/immutable variables	6
[N‑29]	Non-library/interface files should use fixed compiler versions, not floating ones	1
[N‑30]	Typos	5
[N‑31]	File is missing NatSpec	25
[N‑32]	NatSpec @param is missing	135
[N‑33]	NatSpec @return argument is missing	85
[N‑34]	Avoid the use of sensitive terms	5
[N‑35]	Function ordering does not follow the Solidity style guide	36
[N‑36]	Contract does not follow the Solidity style guide's suggested layout ordering	4
[N‑37]	Strings should use double quotes rather than single quotes	1
[N‑38]	Expressions for constant values such as a call to keccak256(), should use immutable rather than constant	51
[N‑39]	Numeric values having to do with time should use time units for readability	9
[N‑40]	Consider using delete rather than assigning zero/false to clear values	5
[N‑41]	Contracts should have full test coverage	1
[N‑42]	Large or complicated code bases should implement invariant tests	