Set the value of the `lotItem.bids[msg.sender]` directly to zero instead to subtracting the full balance, which will give zero always.
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L128
```
lotItem.bids[msg.sender] -= withdrawalAmount;
```