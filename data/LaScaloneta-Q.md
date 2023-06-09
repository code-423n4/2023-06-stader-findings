# LOW

## L-01 Validations on `disableERInspectionMode()` can be bypassed

The access control validations on `disableERInspectionMode()` can be bypassed for anyone when `erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY <= block.number`.

They are also always bypassed for the manager role.

This is due to the `&&` check combining both conditions, instead of evaluating them independently:

```solidity
    function disableERInspectionMode() public override whenNotPaused {
        if (
            !staderConfig.onlyManagerRole(msg.sender) &&
            erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY > block.number
        ) {
            revert CooldownNotComplete();
        }
        erInspectionMode = false;
    }
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L185-L193)

### Impact

Any user can disable `erInspectionMode` when `erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY <= block.number`.

Also, the manager can disable it, regarding of the expected condition.

### Recommended Mitigation Steps

Divide the `&&` statement into two statemens:

```solidity
    function disableERInspectionMode() public override whenNotPaused {
        if (
            !staderConfig.onlyManagerRole(msg.sender)
        ) {
            revert NotManager();
        }
        if (
            erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY > block.number
        ) {
            revert CooldownNotComplete();
        }
        erInspectionMode = false;
    }
```


## L-02 `createLot()` can be spammed with `0` amount lots

`Action::createLot()` has no access control, and allows the creation of lots with `_sdAmount == 0`, with no other extra cost than gas. It allows an adversary to spam zero-amount events for lot creation

```solidity
    function createLot(uint256 _sdAmount) external override whenNotPaused {
        lots[nextLot].startBlock = block.number;
        lots[nextLot].endBlock = block.number + duration;
        lots[nextLot].sdAmount = _sdAmount; // @audit

        LotItem storage lotItem = lots[nextLot];

        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }
        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
        nextLot++;
    }
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L48-L60)

### Recommended Mitigation Steps

Add a check `_sdAmount > 0`

## L-03 Missing `_disableInitializers` on `VaultProxy`
Implementation `VaultProxy` doenst have a `_disableInitializers`, this means that the contract implementation can be initialized:
[VaultProxy.sol#L17](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L17)


## L-04 `addBid` does not increment the `endBlock` of the auction when it is close to the end, preventing the protocol from capturing extra value

When an Auction is created, it sets a `lotItem.endBlock`. This value remains unalterable.

This incentives users to place a bid via `Auction::addBid()`, on the last possible block, as it does not perform any increment on the `lotItem.endBlock`.

```solidity
    function addBid(uint256 lotId) external payable override whenNotPaused {
        // reject payments of 0 ETH
        if (msg.value == 0) revert InSufficientETH();

        LotItem storage lotItem = lots[lotId];
        if (block.number > lotItem.endBlock) revert AuctionEnded();

        uint256 totalUserBid = lotItem.bids[msg.sender] + msg.value;

        if (totalUserBid < lotItem.highestBidAmount + bidIncrement) revert InSufficientBid();

        lotItem.highestBidder = msg.sender;
        lotItem.highestBidAmount = totalUserBid;
        lotItem.bids[msg.sender] = totalUserBid;

        emit BidPlaced(lotId, msg.sender, totalUserBid);
    }
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L62-L78)

### Impact

This prevents the protocol from capturing more value on last minute bids, which is common practive

It discourages earlier participation, and encourages bidders to rather spend more on gas fees to place the bid on the last possible block, rather than providing a bigger bid that will result in more value to the protocol.

### Recommended Mitigation Steps

Add some extra blocks to the `lotItem.endBlock` if there is a bid when the auction is close to its end.

## L-05 Loss of precision due to an issue of dividing first, then multiplying.

On [SDCollateral.sol#L175-L176](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L175-L176):
```
_minSDToBond = convertETHToSD(poolThreshold.minThreshold);
_minSDToBond *= _numValidator;
```

You have a loss of precision because `convertETHToSD` is multiplying then dividing, and this is used to multiply again, allowing a loss of precision.



## L-06 Unsafe use of `trySub`

`trySub()` returns the subtraction of two unsigned integers, with an overflow flag:

```solidity
    function trySub(uint256 a, uint256 b) internal pure returns (bool, uint256) {
        unchecked {
            if (b > a) return (false, 0);
            return (true, a - b);
        }
    }
```

[Link to code](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/Math.sol#L29-L39)

But the overflow value is not checked on the code:

```solidity
(, uint256 remainingPoolTarget) = SafeMath.trySub(poolTotalTarget, currentActiveValidators);
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L62)

```solidity
        (, uint256 availableETHForNewDeposit) = SafeMath.trySub(
            address(this).balance,
            IUserWithdrawalManager(staderConfig.getUserWithdrawManager()).ethRequestedForWithdraw()
        );
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L188-L191)

```solidity
        (, uint256 availableETHForNewDeposit) = SafeMath.trySub(
            address(this).balance,
            IUserWithdrawalManager(staderConfig.getUserWithdrawManager()).ethRequestedForWithdraw()
        );
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L220-L223)

### Recommended Mitigation Steps

Check the returned overflow value to decide the execution path of the function, rather than relying on its side effect of returning a result of 0 on overflow.

## L-07 `ValidatorStatus` uses `INITIALIZED` as its default value

The first value of an enum is its default value. In the case of `ValidatorStatus`, it is `INITIALIZED`.

This can be problematic, as any unitialized value will have by default the `INITIALIZED` status.

`INITIALIZED` is checked, for example in `onlyInitializedValidator()`:

```solidity
    function onlyInitializedValidator(uint256 _validatorId) internal view {
        if (_validatorId == 0 || validatorRegistry[_validatorId].status != ValidatorStatus.INITIALIZED) {
            revert UNEXPECTED_STATUS();
        }
    }
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L712-L716)

```solidity
enum ValidatorStatus {
    INITIALIZED, // @audit default value
    INVALID_SIGNATURE,
    FRONT_RUN,
    PRE_DEPOSIT,
    DEPOSITED,
    WITHDRAWN
}
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/ValidatorStatus.sol#L4-L11)

### Recommended Mitigation Steps

Add an `UNITIALIZED` value to the start of `ValidatorStatus`, so that it is its default value.



# NC
## NC-01 Use `sendValue` instead of raw call

There is a crafted function to send ether, use this instead of raw `.call`.

Use `sendValue` in;
- [UserWithdrawalManager.sol/#L229-L232](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol/#L229-L232)
- [StaderStakePoolsManager.sol/#L102-L105](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol/#L102-L105)
- [StaderInsuranceFund.sol/#L48-L51](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol/#L48-L51)
- [SocializingPool.sol/#L121-L124](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol/#L121-L124)
- [Auction.sol/#L130-L132](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol/#L130-L132)

## NC-02 Grammar error use `initialize` instead of `initialise`
Review this lines:
```
contracts/factory/VaultFactory.sol:42:        VaultProxy(payable(withdrawVaultAddress)).initialise(true, _poolId, _validatorId, address(staderConfig));
contracts/factory/VaultFactory.sol:55:        VaultProxy(payable(nodeELRewardVaultAddress)).initialise(false, _poolId, _operatorId, address(staderConfig));
contracts/VaultProxy.sol:19:    //initialise the vault proxy with data
contracts/VaultProxy.sol:20:    function initialise(
```

## NC-03 Avoid cast multiple time the same variable, example [UtilLib.sol/#L100-L102](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol/#L100-L102)
Instead of
```
address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);
(, , , , , uint256 operatorId, , ) = INodeRegistry(nodeRegistry).validatorRegistry(_validatorId);
(, , , , address operatorAddress) = INodeRegistry(nodeRegistry).operatorStructById(operatorId);
```
Use
```
address nodeRegistry = INodeRegistry(IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId));
(, , , , , uint256 operatorId, , ) = nodeRegistry.validatorRegistry(_validatorId);
(, , , , address operatorAddress) = nodeRegistry.operatorStructById(operatorId);
```

## NC-04 `sendValue` function is duplicated
On [UserWithdrawalManager.sol/#L223-L233](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol/#L223-L233) the `sendValue` function is doing the same as [UtilLib:sendValue](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol/#L167-L172) remove this function and use the `UtilLib.sendValue`

## NC-05 There is no need to cast to `payable` to transfer ETH
Example:
On [Auction.sol/#L131](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol/#L131)
Instead of;
`(bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');`
use
`(bool success, ) = msg.sender.call{value: withdrawalAmount}('');`

## NC-06 Use named import instead of raw imports

## NC-07 There is no need to cast to `payable` to transfer ERC20 Tokens
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L68

-----










## NC-08 `AuctionContract` does not respect the contract convention

It should be `keccak256('AUCTION_CONTRACT')` to respect the convention of the rest of the constants in the contract.

```solidity
    bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');
    bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');
    bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');
    bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract'); // @audit
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L46)


## NC-09 Access control for `markValidatorSettled()` is hidden within a util lib function

`Penalty::markValidatorSettled()` is an important function that resets the `totalPenaltyAmount` of a public key.

It checks its access control permissions through the `UtilLib::getPubkeyForValidSender()` function.

`getPubkeyForValidSender()` is a getter function and should not be the one responsible for access control logic.

Separation of concerns is important for mainteinability and security of the codebase.

It is also worth noting that this function is only used by `markValidatorSettled()`, and no other function.

```solidity
// File: Penalty.sol
    function markValidatorSettled(uint8 _poolId, uint256 _validatorId) external override {
        bytes memory pubkey = UtilLib.getPubkeyForValidSender(_poolId, _validatorId, msg.sender, staderConfig); // @audit
        totalPenaltyAmount[pubkey] = 0;
        emit ValidatorMarkedAsSettled(pubkey);
    }
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L144-L148)

```solidity
// File: UtilLib.sol
    function getPubkeyForValidSender(
        uint8 _poolId,
        uint256 _validatorId,
        address _addr,
        IStaderConfig _staderConfig
    ) internal view returns (bytes memory) {
        address nodeRegistry = IPoolUtils(_staderConfig.getPoolUtils()).getNodeRegistry(_poolId);
        (, bytes memory pubkey, , , address withdrawVaultAddress, , , ) = INodeRegistry(nodeRegistry).validatorRegistry(
            _validatorId
        );
        if (_addr != withdrawVaultAddress) { // @audit
            revert CallerNotWithdrawVault();
        }
        return pubkey;
    }
```

[Link to code](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L49-L63)

### Recommended Mitigation Steps

Return `withdrawVaultAddress` and perform the access control validation on `markValidatorSettled()`