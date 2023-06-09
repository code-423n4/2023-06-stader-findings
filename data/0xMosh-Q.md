# No `amount==O` checks in the `createLot` function in `Auction.sol`

In `Auction.sol` contract , there's no `amount==O` checks to validate the given input . Thus, users can create lot of 0 SD Tokens which is not preferred . 
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L48

# Recommendation 
1. Add a require statement to the `createLot` function, or 
2. Put a if statement and add a custom revert .

