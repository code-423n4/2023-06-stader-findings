The `createLot` method of the `Auction` contract does not check whether the value of `_sdAmount` is equal to 0, because an auction with a quantity of 0 is meaningless

link: https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L48