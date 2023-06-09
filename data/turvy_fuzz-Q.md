## L-1 Manager can disable erInspectionMode when it's Cooldown is not completed
## Summary
use of wrong operator

## Vulnerability Details
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L187
Due to the use of the wrong operator (&& instead of ||), manager can still disable `erInspection Mode` even when it's Cooldown is not completed. Also when not in cooldown, absolutely anyone can still disable it due to the && operator

## Recommendation:
use || instead of &&.

## L-2 Use a 2-step ownership transfer pattern
It’s possible that the `onlyOwner` role mistakenly transfers ownership to a wrong address, resulting in a loss of the `onlyOwner` role.
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L70

## Recommendation
Consider implementing an `acceptOwnership()` function which is called by the `pendingOwner` to confirm the transfer.