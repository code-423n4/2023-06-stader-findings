## [G-01]

No Check on `_sdAmount` When Creating a Lot Results in Gas Loss When `claimSD` & `extractNonBidSD` Are Called (assuming no one will bid on a zero amount of SD tokens).

## Details

Since there's no check on the `_sdAmount` in the `createLot` function; anyone can create a lot with `_sdAmount=0`; thus resulting in gas being wasted on updating variables and emitting events when `claimSD` & `extractNonBidSD` are called.

## Proof of Concept

Instances: 1

```solidity
File: 2023-06-stader/contracts/Auction.sol
Line 51: lots[nextLot].sdAmount = _sdAmount;
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Check if `_sdAmount > 0` in the `createLot` function before creating a lot.

#

## [G-02]

Return The Calculated `newExchangeRate` Variable Directly

## Details

The calculated value of `newExchangeRate` can be returned directly without saving it.

## Proof of Concept

Instances: 1

```solidity
File: 2023-06-stader/contracts/library/UtilLib.sol
Line 161-164:
uint256 newExchangeRate = (totalETHBalance == 0 || totalETHXSupply == 0)
            ? DECIMALS
            : (totalETHBalance * DECIMALS) / totalETHXSupply;
        return newExchangeRate;
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Update the `computeExchangeRate` function to return the result of the calculation directly :

```solidity
return (totalETHBalance == 0 || totalETHXSupply == 0)
? DECIMALS
: (totalETHBalance * DECIMALS) / totalETHXSupply;
return newExchangeRate;
```

#

## [G-03]

`stakeUserETHToBeaconChain()` Should Revert If The Calculated `requiredValidators=0`.

## Details

In `PermissionedPool` & `PermissionlessPool` contracts: when the `STAKE_POOL_MANAGER` calls `stakeUserETHToBeaconChain()` function to deposite ethers for validators: if the sent ethers (msg.value) is less than the amount required to deposite for one validator; this leads to `requiredValidators=0`

## Proof of Concept

Instances: 2

```solidity
File:2023-06-stader/contracts/PermissionedPool.sol
Line 91: uint256 requiredValidators = msg.value / staderConfig.getStakedEthPerNode();
```

```solidity
File:2023-06-stader/contracts/PermissionlessPool.sol
Line 128: uint256 requiredValidators = msg.value / (staderConfig.getFullDepositSize() - DEPOSIT_NODE_BOND);
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Update the `stakeUserETHToBeaconChain()` function to revert if the `requiredValidators=0`.

#