## Low 001
### Title
No slippage protection in `UserWithdrawalManager#finalizeUserWithdrawalRequest`

### Impact 
Users that requested withdrawal by transferring their ethx may receive less eth than they had anticipated

### Proof of Concept
To convert ethx back to eth, user calls `UserWithdrawalManager#requestWithdraw`. This would transfer the user's ethx to the contract, and then store in `UserWithdrawInfo` the ethxAmount, and the eth value of ethxAmount(called amount).
Then, the user can simply wait till someone calls `finalizeUserWithdrawalRequest`, or can decide to call it himself. This would loop through the queued withdrawals, and update the `userWithdrawRequests[requestId].ethFinalized` to `minEThRequiredToFinalizeRequest`, which is the amount of eth the user would be able to claim.
Looking at this:
```solidity
uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);
```
You would notice that the amount that would be credited to the user is the minimum of the amount calculated when the user requested the withdrawal, and the current eth value of the user's ethxamount.
This means that if the current exchange rate of ethx drops, the user would receive an incredibly lower amount than the amount he expected when he requested the withdrawal.

### Tools Used
Manual Review

### Mitigation Steps
Consider allowing the user to specify the least amount of eth he is expecting when he requestWithdraw. If at the time of finalizing the request, the current eth value of user's ethxAmount is lower than the user's minExpectedAmount, the user request should be moved to another queue to be processed at a later time
