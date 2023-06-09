
# Report

## Summary 

| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | To optimize gas usage, invoke this external function once | 2 |
| [GAS-2](#GAS-2) | It doesn't need to calculate the bids amount. | 1 |

## Issues 


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | To optimize gas usage, invoke this external function once | 2 |
| [GAS-2](#GAS-2) | It doesn't need to calculate the bids amount. | 1 |

### <a name="GAS-1"></a>[GAS-1] To optimize gas usage, invoke this external function once

#### Impact:
To optimize gas usage, invoke this external function once `staderConfig.getStakePoolManager()`, `staderConfig.getStaderTreasury()`

#### Vulnerability Detail


```solidity
File: contracts/Auction.sol:L102-L103
    IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveEthFromAuction{value: ethAmount}();
    emit ETHClaimed(lotId, staderConfig.getStakePoolManager(), ethAmount);

File: contracts/Auction.sol:L114-L117
    if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {
        revert SDTransferFailed();
    }
    emit UnsuccessfulSDAuctionExtracted(lotId, _sdAmount, staderConfig.getStaderTreasury());
```


#### Recommendation

Recommend that declare the local variable to save address and use it.

### <a name="GAS-2"></a>[GAS-2] It doesn't need to calculate the bids amount.

#### Impact:
It doesn't need to calculate the bids amount.

#### Vulnerability Detail


```solidity
File: contracts/Auction.sol:L128
    lotItem.bids[msg.sender] -= withdrawalAmount; 

File: contracts/Auction.sol:L128
    lotItem.bids[msg.sender] = 0; 
```


#### Recommendation

Recommend to directly set it to 0.






