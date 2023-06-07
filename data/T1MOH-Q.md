## Allow bidders to withdraw not highest bid when auction is not ended
[File: contracts/Auction.sol](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L122)

Remove the check that the auction has ended. It blocks users' funds and doesn't make any sense
```solidity
    function withdrawUnselectedBid(uint256 lotId) external override nonReentrant {
        LotItem storage lotItem = lots[lotId];
        if (block.number <= lotItem.endBlock) revert AuctionNotEnded(); //@audit
        if (msg.sender == lotItem.highestBidder) revert BidWasSuccessful();

        ...
    }
```