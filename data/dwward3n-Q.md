- `Auction::createLot` can be called by anyone.
Consider adding a check `require(_sdAmount > 0)` in `createLot`. This will be a check for valid lot.
Also, in `extractNonBidSD`, `sdAmount` is used for checking already claimed so it's also good for consistent semantics.

- For consistency, remove `vaultSettleStatus` in `ValidatorWithdrawalVault`.
Both vaults, `ValidatorWithdrawalVault` and `NodeELRewardVault` should be stateless since their state will be in VaultProxy.
To reduce confusion, remove `vaultSettleStatus` in `ValidatorWithdrawalVault` and instead add a setter in VaultProxy and call it.