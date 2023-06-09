[L-01] value of shares minted in `deposit()` not check in `StaderStakePoolsManager.sol`


# Links to affected code:
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L174


## Impact
The value of share minted is not check in `StaderStakePoolsManager.sol`, It is recommended to check that the share minted is not equal to 0.



## Proof of Concept
```
function deposit(address _receiver) public payable override whenNotPaused returns (uint256) {
        uint256 assets = msg.value; //10**14 = 0.0004 ETH
        if (assets > maxDeposit() || assets < minDeposit()) { 
            revert InvalidDepositAmount();
        }
        uint256 shares = previewDeposit(assets); //@audit-issue Does not check share != 0
        _deposit(msg.sender, _receiver, assets, shares);
        return shares;
    }
```

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Add a require statement as follow in StaderStakePoolsManager.sol#L175:

```
function deposit(address _receiver) public payable override whenNotPaused returns (uint256) {
        uint256 assets = msg.value; //10**14 = 0.0004 ETH
        if (assets > maxDeposit() || assets < minDeposit()) { 
            revert InvalidDepositAmount();
        }
        uint256 shares = previewDeposit(assets);
    ++  require(shares != 0, "Should not mint 0 shares.");
        _deposit(msg.sender, _receiver, assets, shares);
        return shares;
    }

```
