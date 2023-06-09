## On [createLot](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L48) function on auction.sol `nextLot` can be cached `5` times.
```diff
1,5c1,4
<     function createLot(uint256 _sdAmount) external override whenNotPaused {
<         uint256 currentLot = nextLot;
<         lots[currentLot].startBlock = block.number;
<         lots[currentLot].endBlock2R8xVW3WiFqzxw = block.number + duration;
<         lots[currentLot].sdAmount = _sdAmount;
---
>    function createLot(uint256 _sdAmount) external override whenNotPaused {
>         lots[nextLot].startBlock = block.number;
>         lots[nextLot].endBlock = block.number + duration;
>         lots[nextLot].sdAmount = _sdAmount;
7c6
<         LotItem storage lotItem = lots[currentLot];
---
>         LotItem storage lotItem = lots[nextLot];
12c11
<         emit LotCreated(currentLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
---
>         emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
15d13
<
```
Saves `512` gas
`before:` [PASS] test_createLot() (gas: 178563)
`after:`[PASS] test_createLot() (gas: 178051)