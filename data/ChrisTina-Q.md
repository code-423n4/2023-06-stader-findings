## Function `setVariable` should emit event `SetVariable` instead of `SetConstant`

The function `setVariable` in `StaderConfig.sol` emits the event `SetConstant`. There is also an event `SetVariable` defined in the inferface which would be more descriptive of what the function does.

```solidity
function setVariable(bytes32 key, uint256 val) internal {
  variablesMap[key] = val;
  emit SetConstant(key, val);
}

```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderConfig.sol#L476

```solidity
  // Events
    event SetConstant(bytes32 key, uint256 amount);
    event SetVariable(bytes32 key, uint256 amount);
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/interfaces/IStaderConfig.sol#L12-L14

## State changes after interactions

The function `createLot` in `Auction.sol` does not follow the checks-effects-interactions pattern

```solidity
// token transfer
if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
            revert SDTransferFailed();
}
// emitting event
emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
// state changes
nextLot++;
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L55-L59

## Mismatch between code and documentation

The comments describe the intention of the `withdraw` function in `SDCollateral.sol` as follows:

```solidity
 /// @notice for operator to request withdraw of sd
    /// @dev it does not transfer sd tokens immediately
    /// operator should come back after withdrawal-delay time to claim
    /// this requested sd is subject to slashes
```

but in reality the sd tokens are transfered immediately and do not have to be claimed.

```solidity
function withdraw(uint256 _requestedSD) external override {
 ...
  if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
    revert SDTransferFailed();
  }
  ...
}

```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L54-L73

## Missing override keyword

Not technically necessary, but consider adding for consistency
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderConfig.sol#L361
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderConfig.sol#L466
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderConfig.sol#L500-L502
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L316

## Duplicate event information

Due to the previous check

```solidity
 if (msg.sender != userRequest.owner) {
            revert CallerNotAuthorizedToRedeem();
        }
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/UserWithdrawalManager.sol#L174-L176

it is redundant to list both `msg.sender` and `userRequest.owner`, both will have the same value.

```solidity
emit RequestRedeemed(msg.sender, userRequest.owner, etherToTransfer);
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/UserWithdrawalManager.sol#L180

## Empty constructor could be omitted

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/NodeELRewardVault.sol#L14
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ValidatorWithdrawalVault.sol#L23
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L17

## Unused imports

`INodeRegistry` in `NodeELRewardvault.sol`
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/NodeELRewardVault.sol#L8s
`IPermissionlessNodeRegistry` in `SocializingPool.sol`
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#L9
`Math.sol` is not used in PermissionedPool and PermissionlessPool
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L22
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessPool.sol#L20

## Usage of library functions

Library functions are referenced directly throughout the codebase, so the `using A for B` directive can be omitted.

Example:

```solidity
selectedPoolCapacity = Math.min(
  Math.min(poolAllocationMaxSize, _newValidatorToRegister),
  Math.min(remainingPoolCapacity, remainingPoolTarget)
);

```

```solidity
using Math for uint256;
using SafeMath for uint256;

```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L15-L16
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L63-L66

## Typos / Naming inconsistencies

The spelling "initialise" is used in 3 instances, while the rest of the codebase uses "initialize".
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L19
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L20
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/factory/VaultFactory.sol#L42

Error `OperatorIsDeactivate` => `OperatorIsDeactivated`
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L686

```solidity
 * @dev any one call, permissionless
```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L83
