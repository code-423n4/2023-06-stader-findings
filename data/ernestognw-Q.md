# Excessive use of `DEFAULT_ADMIN_ROLE` introduce unexpected centralization risks

Across the codebase, a total of 19 `_grantRole(DEFAULT_ADMIN_ROLE, _admin)` can be found in the `initialize` functions. This gives the `DEFAULT_ADMIN_ROLE` the ability to not just control the functions marked as `DEFAULT_ADMIN_ROLE` but also every other role in each `AccessControlUpgradeable` inherited since other roles don't define a custom admin by calling `_setRoleAdmin`.

This behavior introduces the following risks:

- Unexpectedly having more than 1 `DEFAULT_ADMIN_ROLE` with privileged access
- Any other role is governed by the `DEFAULT_ADMIN_ROLE`

## Recommendation

It's recommended to use [OpenZeppelin's `AccessControlDefaultAdminRules`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/AccessControlDefaultAdminRulesUpgradeable.sol) and clearly specify each role's admin by explicitly calling `_setRoleAdmin`.

---

# Anyone can spam the `receive*` functions of the `StaderStakePoolsManager`

In the `StaderStakePoolsManager` there are currently 3 functions indicating the contract has received ether. The functions are written as follows:

```solidity
// protection against accidental submissions by calling non-existent function
fallback() external payable {
    revert UnsupportedOperation();
}

// protection against accidental submissions by calling non-existent function
receive() external payable {
    revert UnsupportedOperation();
}

// payable function for receiving execution layer rewards.
function receiveExecutionLayerRewards() external payable override {
    emit ExecutionLayerRewardsReceived(msg.value);
}
```

However, these functions don't have any access control on their definition, so they can be called by anyone in order to spam any indexing service depending on the events emitted.

## Recommendation

Implement Access Control to the `receive*` functions and allowing only known contracts to call these functions.

---

# The initializer in `VaultProxy` should be called `initialize` instead of `initialise`

The function `initialise` in `VaultProxy` is named incorrectly. Although this may not bring any instantiation issue if used correctly, it brings some potential issues:

- The contract will require a new mechanism for reinitialization in case of an upgrade
- Any deployment tooling relying on the name `initialize` will fail or leave the `initialise` open for frontrunning

## Recommendation

[Use OpenZeppelin's `Initializer.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4c713f8cea2cbd002509c8ea0d8e0a996627f069/contracts/proxy/utils/Initializable.sol)

---

# The `fallback` function in `VaultProxy` incorrectly bubbles up revert information

The `fallback()` function in `VaultProxy.sol` makes a `delegatecall` to the vault implementation and it's intended to bubble up the revert reason in case of failure. The current code is:

```solidity
fallback(bytes calldata _input) external payable returns (bytes memory) {
    address vaultImplementation = isValidatorWithdrawalVault
        ? staderConfig.getValidatorWithdrawalVaultImplementation()
        : staderConfig.getNodeELRewardVaultImplementation();
    (bool success, bytes memory data) = vaultImplementation.delegatecall(_input);
    if (!success) {
        revert(string(data));
    }
    return data;
}
```

However, reverting with the returned data may be incorrectly parsed by a caller if there are custom errors used by the implementation (as there is in `NodeELRewardVault.sol` and `ValidatorWithdrawalVault.sol`) because the default encoding would be `keccak256("Error(string)")` where `string` would be the implementation's error selector.

The impact is that any contract interacting with the Vault proxy might expect some error selector when it gets an unexpected encoding.

## Recommendation

Use OpenZeppelin's [Address.verifyCallResult](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4c713f8cea2cbd002509c8ea0d8e0a996627f069/contracts/utils/Address.sol#L181) to ensure correct error bubbling