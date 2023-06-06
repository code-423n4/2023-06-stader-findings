# Informational
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L83

In the claimSD function, the condition if (msg.sender != lotItem.highestBidder) revert **notQualified()**;
```

should be
if (msg.sender != lotItem.highestBidder) revert **NotQualified()**; to match the capitalization of the error message.
