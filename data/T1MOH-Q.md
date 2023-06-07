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

## Add additional check in `createLot()` that msg.sender is SDCollateral.sol
Add this check to prevent users from interacting with system contract and following losing of SD tokens. Because ETH and SD tokens will be transferred to StaderTreasury and StaderStakePoolsManager.sol
```solidity
    function createLot(uint256 _sdAmount) external override whenNotPaused {
        //SOLUTION
        require(msg.sender == staderConfig.getSDCollateral());

        lots[nextLot].startBlock = block.number;
        lots[nextLot].endBlock = block.number + duration;
        lots[nextLot].sdAmount = _sdAmount;

        LotItem storage lotItem = lots[nextLot];

        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }
        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
        nextLot++;
    }
```