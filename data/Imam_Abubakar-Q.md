# No events emitted for minting and burning
## Impact:
Although not a vulnerability, it is a good practice to emit events when tokens are minted or burned. This helps in tracking token movements and provides better transparency.

Proof of Concept:
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ETHx.sol#L47
```
 function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) whenNotPaused {
        _mint(to, amount);
    }
```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ETHx.sol#L56
```
 function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {
        _burn(account, amount);
    }
```

## Tools Used:
Manual code review

## Recommended Mitigation Steps:
Emit events for minting and burning tokens to improve transparency and traceability.