- https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L215
In ```depositETHOverTargetWeight``` function, whenever a call to PoolSelector's ```poolAllocationForExcessETHDeposit``` function is made by PoolManager it changes ```poolIdArrayIndexForExcessDeposit``` in PoolSelector even with as little balance as 1 wei. This can be used by malicious actors, to not allow a any of pool type to be used for depositing excess ETH.

- Lots of places where (a*b)/c is being used which can be replaced with FullMath.mulDiv to protect from overflow while multiplying. Eg. https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L207
