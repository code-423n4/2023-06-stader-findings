1.
Title: empty `constructor`

Proof of Concept:
[NodeELRewardVault.sol#L14](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L14)

Recommended Mitigation Steps:
Remove if unused for gas saving
________________________________________________________________________

2.
Title: Using `msg.sender` directly is more effective

Proof of Concept:
[OperatorRewardsCollector.sol#L47](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L47)
[SDCollateral.sol#L44](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L44)
[SDCollateral.sol#L59](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L59)

Recommended Mitigation Steps:
delete `operator` and use `msg.sender` directly instead of caching it.

```
	uint256 amount = balances[msg.sender];
        balances[msg.sender] -= amount;
```
________________________________________________________________________

3.
Title: function `getTotalQueuedValidatorCount()` gas improvement on returning value

Proof of Concept:
[PermissionedNodeRegistry.sol#L487](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L487)

Recommended Mitigation Steps:
by set `totalQueuedValidators` in returns L#486 and delete L#487 can save gas

```
function getTotalQueuedValidatorCount() external view override returns (uint256 totalQueuedValidators) { //@audit-info: set here
```
________________________________________________________________________

4.
Title: function `getOperatorTotalNonTerminalKeys()` gas improvement on returning value

Proof of Concept:
[PermissionedNodeRegistry.sol#L533](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L533)

Recommended Mitigation Steps:
by set `totalNonWithdrawnKeyCount` in returns L#526 and delete L#533 can save gas

```
function getOperatorTotalNonTerminalKeys(
        address _nodeOperator,
        uint256 _startIndex,
        uint256 _endIndex
    ) public view override returns (uint64 totalNonWithdrawnKeyCount) { //@audit-info: set here
```
________________________________________________________________________

5.
Title: Avoid unnecessary memory allocation and copy

Proof of Concept:
[PermissionlessPool.sol#L276-L286](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L276-L286)

Recommended Mitigation Steps:
instead of allocating a new bytes array and copying the values one by one, you can use `abi.encodePacked` to directly pack the values into a bytes array.

Consider change it to:
```
function to_little_endian_64(uint256 _depositAmount) internal pure returns (bytes memory) {
    uint64 value = uint64(_depositAmount / 1 gwei);
    return abi.encodePacked(value);
}
```
________________________________________________________________________

6.
Title: Avoid unnecessary variable assignments

Proof of Concept:
[PoolUtils.sol#L256-L257](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L256-L257)

Recommended Mitigation Steps:
Instead of assigning `usersETH` and `collateralETH` to variables, use it directly.

Consider change it to:
```
function calculateRewardShare(uint8 _poolId, uint256 _totalRewards)
    external
    view
    override
    returns (
        uint256 userShare,
        uint256 operatorShare,
        uint256 protocolShare
    )
{
    uint256 TOTAL_STAKED_ETH = staderConfig.getStakedEthPerNode();
    uint256 protocolFeeBps = getProtocolFee(_poolId);
    uint256 operatorFeeBps = getOperatorFee(_poolId);

    uint256 _userShareBeforeCommission = (_totalRewards * (TOTAL_STAKED_ETH - getCollateralETH(_poolId))) / TOTAL_STAKED_ETH; //@audit-info: set here

    protocolShare = (protocolFeeBps * _userShareBeforeCommission) / staderConfig.getTotalFee();

    operatorShare = (_totalRewards * getCollateralETH(_poolId)) / TOTAL_STAKED_ETH; //@audit-info: set here
    operatorShare += (operatorFeeBps * _userShareBeforeCommission) / staderConfig.getTotalFee();

    userShare = _totalRewards - protocolShare - operatorShare;
}
```
________________________________________________________________________