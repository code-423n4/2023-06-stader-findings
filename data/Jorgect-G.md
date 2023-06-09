# GAS OPTIMIZATION

## [G-01] Set to zero insted of doing a substract

The contract is doing a substract to set lotItem.bids[msg.sender] to 0, see the next lines in Action contract:

```
file: /contracts/Auction.sol
 function withdrawUnselectedBid(uint256 lotId) external override nonReentrant {
        ...
        uint256 withdrawalAmount = lotItem.bids[msg.sender];
        if (withdrawalAmount == 0) revert InSufficientETH();

        lotItem.bids[msg.sender] -= withdrawalAmount;

        ...
    }
```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L128

Recomentaction:
Set lotItem.bids[msg.sender] to 0, saving around 649 units of gas calculating in remix.

```
file: /contracts/Auction.sol
 function withdrawUnselectedBid(uint256 lotId) external override nonReentrant {
        ...
        uint256 withdrawalAmount = lotItem.bids[msg.sender];
        if (withdrawalAmount == 0) revert InSufficientETH();

        lotItem.bids[msg.sender] = 0;

        ...
    }
```



