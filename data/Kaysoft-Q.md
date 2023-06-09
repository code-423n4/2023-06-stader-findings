## [NC-1] 7200 seconds is assumed to be 24hours.

From the comment in the code below of Auction.sol file, it is assumed that 7200 seconds is 24hours which is false.

```
uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours
```
File: https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L22

## [NC-2] USE 2 STEP OWNERSHIP CHANGE
The `updateOwner` of the VaultProxy.sol is used to change the owner in one transaction. Use two step owner ship change like it is done in Openzeppelin's ownership2step contract.

```
function updateOwner(address _owner) external override onlyOwner {
        UtilLib.checkNonZeroAddress(_owner);
        owner = _owner;
        emit UpdatedOwner(owner);
    }
```
File: https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L68