Administrative functions in the ETHx contract are protected by role-based authentication. The roles are mostly consistent, however, the pause and unpause functions stick out:

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ETHx.sol#L64-L75

```
    function pause() external {  // @audit-issue (low) inconsistent authorization (esp. pause/unpause)
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        _pause();
    }

    /**
     * @dev Returns to normal state.
     * Contract must be paused
     */
    function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) {
        _unpause();
    }
```

I recommend either moving both functions to the `DEFAULT_ADMIN_ROLE`, or delegating both authorization decisions to the `staderConfig` manager value.