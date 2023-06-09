## [QA-01]

Lack of a Double-Step TransferOwnership Pattern

## Details

In VaultProxy.sol contract; `updateOwner` function:
It is recommended to implement a two-step process where the owner nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of the ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

## Impact

This can lead to transfer contract ownership to a dormant/compromised/non-existent contract which leads to disable all the functions that are only accessible by the owner.

## Proof of Concept

Instances: 1

```solidity
File: 2023-06-stader/contracts/VaultProxy.sol
Line 68: function updateOwner(address _owner) external override onlyOwner {
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

This can be easily achieved by using OpenZeppelin’s Ownable2Step contract.

#

## [QA-02]

`MIN_AUCTION_DURATION` Is a Hardcoded Value While It might Change Depending on The Network.

## Details

In `Auction.sol` contract; the `MIN_AUCTION_DURATION` variable is set to 7200 which is supposed to be the minimum number of blocks minted on Ethereum blockchain in one day; but in fact this number might be decreased/increased drastically as per the current incidents taht happened recently/or expected future upgrades.

## Impact

- When updating the state variable `duration`; the `updateDuration` function checks the input with the hardcoded `MIN_AUCTION_DURATION`.
- If the network in the future has been upgarded to a number of minted blocks extremely larger than the current hardcoded minimum duration (say 100_000 transaction/second); this will result in crating lots and ending them in a very short time without having any bidders on them.

## Proof of Concept

Instances: 1

```solidity
File: 2023-06-stader/contracts/Auction.sol
Line 22: uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours
Line 38: duration = 2 * MIN_AUCTION_DURATION;
Line 50:lots[nextLot].endBlock = block.number + duration;
Line 67: if (block.number > lotItem.endBlock) revert AuctionEnded();
Line 82: if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
Line 97: if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
Line 108: if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
Line 122: if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
Line 146: if (_duration < MIN_AUCTION_DURATION) revert ShortDuration();
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

- Since the `Auction` contract can be initialized multiple times: setting `MIN_AUCTION_DURATION` upon contract initialization instead of being a constant/hardcoded.

#

## [QA-03]

`lots[lotId].highestBidder` Can't Withdraw From a Bid as The Rest of The Bidders

## Details

## Proof of Concept

Instances: 1

```solidity
File: 2023-06-stader/contracts/Auction.sol
Line 123: if (msg.sender == lotItem.highestBidder) revert BidWasSuccessful();
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Add a function that enables the `lots[lotId].highestBidder` from withdrawing from a bid.

#

## [QA-04]

`lotItem.sdAmount` is Not Updated to Zero in `claimSD` Function.

## Details

- When `claimSD` function is invoked; it updates `lotItem.sdClaimed = true` and doesn't update `lotItem.sdAmount=0`.
- But when `extractNonBidSD` is invoked; it checks `if (lotItem.sdAmount == 0) revert AlreadyClaimed();` instead of checking `if (lotItem.sdClaimed = true) revert AlreadyClaimed();`

## Impact

- Since there's no check on `_sdAmount` if > 0 when creating a lot; this will result in emitting a wrong error `AlreadyClaimed()` when calling `extractNonBidSD` for a lot with `lotItem.sdAmount == 0` while this is not the case!

## Proof of Concept

Instances: 1

```solidity
File: 2023-06-stader/contracts/Auction.sol
Line 80: function claimSD(uint256 lotId) external override {
Line 110: if (lotItem.sdAmount == 0) revert AlreadyClaimed();
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Update `extractNonBidSD` function to check `if (lotItem.sdClaimed == true) revert AlreadyClaimed();` instead of checking `if (lotItem.sdAmount == 0) revert AlreadyClaimed();`

#

## [QA-05]

If a Non-Exitent Function Is Called With a `msg.value`, The Sender Will Not Be Refunded.

## Details

- `NodeELRewardVault` contract is callable via `VaultProxy` fallback function via low-level `delegatecall` , if by mistake a `delegatecall` is made to the `NodeELRewardVault` contract with value; the `msg.value` will be transferred un-intentionally to the contract.

- Same issue is spotted in `ValidatorWithdrawalVault` contract.

- Almost a similar issue is spotted in `SocializingPool` &`UserWithdrawalManager`contracts (but without the low-level `delegatecall` redirecting to the contracts)

## Impact

- If by mistake a call is made to any of the above mentioned contracts to a non-existent function with Ethers being sent with the call ; the `msg.value` will be transferred un-intentionally to the called contract.

## Proof of Concept

Instances: 4

```solidity
File:2023-06-stader/contracts/NodeELRewardVault.sol
Line 20-22:
 receive() external payable {
        emit ETHReceived(msg.sender, msg.value);
    }
```

```solidity
File:2023-06-stader/contracts/ValidatorWithdrawalVault.sol
Line 26-28:
 receive() external payable {
        emit ETHReceived(msg.sender, msg.value);
    }
```

```solidity
File:2023-06-stader/contracts/SocializingPool.sol
Line 57-59:
 receive() external payable {
        emit ETHReceived(msg.sender, msg.value);
    }
```

```solidity
File:2023-06-stader/contracts/UserWithdrawalManager.sol
Line 68-70:
 receive() external payable {
        emit ETHReceived(msg.sender, msg.value);
    }
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

- Add a separate function to receive funds instead of using the built-in receive function:

```solidity
function  receiveFunds() external payable {
        emit ETHReceived(msg.sender, msg.value);
    }
```

- Change current receive function to revert with an error if a non-existent function in the contract is called:

```solidity
function  receive external payable {
        revert NonExistentFunctionCalled();
    }
```

- The above changes will require changing all the instances in the project contracts from sending ETHs to the above mentioned contracts from using `.call{value:fundsValue}` into using `.receiveFunds{value:fundsValue}()`.

#

## [QA-06]

Operator Can Register Validators With The Same `_preDepositSignature` & `_depositSignature`

## Details

When the operator adds validators via `addValidatorKeys` function in the `PermissionlessNodeRegistry` & `PermissionedNodeRegistry` contracts; he can pass `_preDepositSignature` & `_depositSignature` arrays with the same signatures for all the pubKey of the validators; and this will pass since the validation is made only on the length of the entries of the two `_preDepositSignature` & `_depositSignature`.

## Proof of Concept

Instances: 2

```solidity
File: 2023-06-stader/contracts/PermissionlessNodeRegistry.sol
Line 141:
IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);
```

```solidity
File: 2023-06-stader/contracts/PermissionedNodeRegistry.sol
Line 156:
IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i]);
```

```solidity
File: 2023-06-stader/contracts/PoolUtils.sol
Line 230-238:
  if (_pubkey.length != PUBKEY_LENGTH) {
            revert InvalidLengthOfPubkey();
        }
        if (_preDepositSignature.length != SIGNATURE_LENGTH) {
            revert InvalidLengthOfSignature();
        }
        if (_depositSignature.length != SIGNATURE_LENGTH) {
            revert InvalidLengthOfSignature();
        }
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Check if the preDepositSignature & depositSignature have already been used/assigned for other validators before adding a new validator (following the same check used in `isExistingPubkey` function in the `PoolUtils` contract)

#

## [QA-07]

`_verifiedKeyBatchSize` Input To Be Checked If >=3 Before Updating The State Variable `verifiedKeyBatchSize`.

## Details

In `PermissionedNodeRegistry.sol`: when updating the `verifiedKeyBatchSize` in `updateVerifiedKeysBatchSize` function; the `_verifiedKeysBatchSize` is not checked if `>=3` before assigning it to the state variable.

## Impact

If the value of `verifiedKeysBatchSize` is updated to a value of 0 ; this will disable the operator from using `markValidatorReadyToDeposit` that changes the validators states, since it will revert due to this condition:

```solidity
function markValidatorReadyToDeposit(
        bytes[] calldata _readyToDepositPubkey,
        bytes[] calldata _frontRunPubkey,
        bytes[] calldata _invalidSignaturePubkey
    ) external override nonReentrant whenNotPaused {
    //.......
  if (
            readyToDepositValidatorsLength + frontRunValidatorsLength + invalidSignatureValidatorsLength >
            verifiedKeyBatchSize
        ) {
            revert TooManyVerifiedKeysReported();
        }
    //....... rest of the code
    }
```

- Why `_verifiedKeysBatchSize >= 3` ?
  3 is selected here to make the `markValidatorReadyToDeposit` function capable of handling the three mentioned operations for at least one validotor sent via each array (\_readyToDepositPubkey,\_frontRunPubkey,\_invalidSignaturePubkey).

## Proof of Concept

Instances: 1

```solidity
File: 2023-06-stader/contracts/PermissionlessNodeRegistry.sol
Line 440: verifiedKeyBatchSize = _verifiedKeysBatchSize;
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Check if the input value `_verifiedKeysBatchSize >= 3` before assigning it to the `verifiedKeyBatchSize` state variable.

#

## [QA-08]

Extra Ethers Sent By The `STAKE_POOL_MANAGER` to Deposit For Validators Are Not Refunded Back.

## Details

In `PermissionedPool` & `PermissionlessPool` contracts: when the `STAKE_POOL_MANAGER` calls `stakeUserETHToBeaconChain()` function to deposit ethers for validators: if the sent ethers (msg.value) is greater than the amount required to deposite for an integer number of validators (i.e `msg.value % deposite required for one validator != 0`); the remaining `msg.value` will not be sent back to the pool manager.

## Impact

If the `STAKE_POOL_MANAGER` sends extra ethers to deposit for validators or sends not enough ethers to deposite for one validator (i.e `requiredValidators=0`); he will not be refunded the sent ethers or the extra ethers.

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

- Revert the `stakeUserETHToBeaconChain()` function if the `requiredValidators=0`.
- Refund the sent extra ethers to the pool manager.
