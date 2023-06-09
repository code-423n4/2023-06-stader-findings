### [Gas-01] Function for creating `lot` i.e `createLot` can be more gas efficient as follows


```solidity
-        lots[nextLot].startBlock = block.number; // @audit gas efficient way mentioned in RemixId
-        lots[nextLot].endBlock = block.number + duration;
-        lots[nextLot].sdAmount = _sdAmount;

        LotItem storage lotItem = lots[nextLot];
```        
Above could replace with
```solidity
         LotItem storage lotItem = lots[nextLot]

+        lotItem.startBlock = block.number;
+        lotItem.endBlock = block.number + duration;
+        lotItem.sdAmount = _sdAmount;
```

First one cost :: 130290 gas 
Second one cost :: 110009 gas (NB :: I only mentioned here gas used to only store value to state variable)

*Instances(1)*
```
File: contracts/Auction.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L49-L51
```


### [Gas-02] `lotItem` variable has no significance in `createLot()` function, so it should be removed


*Instances(1)*
```
File: contracts/Auction.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L53
```


### [Gas-03] `lotItem.bids[msg.sender]` could set to `0` without arithmatic operation due to above check 
as `withdrawalAmount` nothing but `lotItem.bids[msg.sender]`, so `lotItem.bids[msg.sender]` can directly set to `0`
```solidity
    function withdrawUnselectedBid(uint256 lotId) external override nonReentrant {
        LotItem storage lotItem = lots[lotId];
        if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
        if (msg.sender == lotItem.highestBidder) revert BidWasSuccessful();

        uint256 withdrawalAmount = lotItem.bids[msg.sender];
        if (withdrawalAmount == 0) revert InSufficientETH();

-       lotItem.bids[msg.sender] -= withdrawalAmount;
+       lotItem.bids[msg.sender] = 0;
```

*Instances(1)*
```
File: contracts/Auction.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L128
```

### [Gas-04] For transfering `ETH` use of assembly code can be more gas efficient 
```solidity
-        (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}(''); 
-        if (!success) revert ETHWithdrawFailed();

+        bool succ;
+        assembly{
+            succ := call(gas(), msg.sender, withdrawalAmount, 0, 0)
+        }
```
*Instances(1)*

```
File: contracts/Auction.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L131-L132
```



### [Gas-05] Zero address can perform in a function, no need for extra `JUMP` (i.e for checking just zero address function calling another contract function) which is more gas costlier

You can simply perform zero address check in `updateStaderConfig()` function, even with `assembly` this will be more gas efficient, no need to call external contract.
```solidity
    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
+        require(_staderConfig != address(0), 'BOOM BOOM');
-        UtilLib.checkNonZeroAddress(_staderConfig); 
         staderConfig = IStaderConfig(_staderConfig);
         emit UpdatedStaderConfig(_staderConfig);
    }
```

*Instances(1)*
```
File: contracts/ETHx.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L86
```

### [Gas-06] Instead of `address(this).balance`, `self.balance()` should be used, which is more gas efficient

*Instances(7)*
```
File: contracts/NodeELRewardVault.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L28
```
```
File: contracts/ValidatorWithdrawalVault.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L34
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L99
```
```
File: contracts/StaderInsuranceFund.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L43
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L62
```


### [Gas-07] No need to cache `msg.sender` below
```solidity
    function claim() external whenNotPaused {
-        address operator = msg.sender; // @audit G=> can optimized to one line.
-        uint256 amount = balances[operator];
-        balances[operator] -= amount;

-        uint256 amount = balances[msg.sender];
-        balances[msg.sender] -= amount;
....
....
```

*Instances(1)*
```
File: contracts/OperatorRewardsCollector.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L46-L49
```

### [Gas-08] uint storage overhead
```solidity
uint16 public poolAllocationMaxSize;
```
```solidity
mapping(uint8 => uint256) public poolWeights;
```

*Instances(4)*
```
File:: contracts/PoolSelector.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L18
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L23
```
```
File:: contracts/PoolUtils.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L17
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L18
```


### [Gas-09] ```address nodeRegistry``` should create outside of the loop and get overrided with very iteration of the loop, which will be more gas efficient. 
```solidity
    function isExistingPubkey(bytes calldata _pubkey) public view override returns (bool) { 
        uint256 poolCount = getPoolCount();
        for (uint256 i = 0; i < poolCount; i++) {
            address nodeRegistry = getNodeRegistry(poolIdArray[i]);
            if (INodeRegistry(nodeRegistry).isExistingPubkey(_pubkey)) {
                return true;
            }
        }
        return false;
    }
```

*Instances(4)*
```
File:: contracts/PoolUtils.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L169
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L180
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L191
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L202
```

### [Gas-10] 4 functions could be enclosed to 1 function with 4 return value
As `isExistingPubkey()`, `isExistingOperator()`, `getOperatorPoolId()` and `getValidatorPoolId` working, validating and calling same external contract similar manner so these could arranged in a single function with 4 return value which will save some deployment gas cost.

```solidity
    function isExistingPubkey(bytes calldata _pubkey) public view override returns (bool) { // @audit G:: These 4 function can perform in one function with 4 return value
        uint256 poolCount = getPoolCount();
        for (uint256 i = 0; i < poolCount; i++) {
            address nodeRegistry = getNodeRegistry(poolIdArray[i]); // @audit G:: This variable should created outside of the loop
            if (INodeRegistry(nodeRegistry).isExistingPubkey(_pubkey)) {
                return true;
            }
        }
        return false;
    }

    function isExistingOperator(address _operAddr) external view override returns (bool) {
        uint256 poolCount = getPoolCount();
        for (uint256 i = 0; i < poolCount; i++) {
            address nodeRegistry = getNodeRegistry(poolIdArray[i]); // @audit G:: This variable should created outside of the loop
            if (INodeRegistry(nodeRegistry).isExistingOperator(_operAddr)) {
                return true;
            }
        }
        return false;
    }

    function getOperatorPoolId(address _operAddr) external view override returns (uint8) {
        uint256 poolCount = getPoolCount();
        for (uint256 i = 0; i < poolCount; i++) {
            address nodeRegistry = getNodeRegistry(poolIdArray[i]);  // @audit G:: This variable should created outside of the loop
            if (INodeRegistry(nodeRegistry).isExistingOperator(_operAddr)) {
                return poolIdArray[i];
            }
        }
        revert OperatorIsNotOnboarded();
    }

    function getValidatorPoolId(bytes calldata _pubkey) external view override returns (uint8) {
        uint256 poolCount = getPoolCount();
        for (uint256 i = 0; i < poolCount; i++) {
            address nodeRegistry = getNodeRegistry(poolIdArray[i]);  // @audit G:: This variable should created outside of the loop
            if (INodeRegistry(nodeRegistry).isExistingPubkey(_pubkey)) {
                return poolIdArray[i];
            }
        }
        revert PubkeyDoesNotExit();
    }

```
*Instances(1)*
```
File:: contracts/PoolUtils.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L166-L209
```

### [Gas-11] Inherite `Openzeppelin Initializable library` which will be more gas efficient than your custom one
```solidity
...........
...........
   bool public override isInitialized;
...........
...........
   function initialise( 
        bool _isValidatorWithdrawalVault,
        uint8 _poolId,
        uint256 _id,
        address _staderConfig
    ) external {
        if (isInitialized) {
            revert AlreadyInitialized();
        }
............
............       
```
*Instances(1)*
```
File:: contracts/VaultProxy.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L26-L28
```
### [Gas-12] Openzeppelin's `safeTransferFrom()` should used which is more gas efficient and give more accurate error.
```solidity
if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) { 
            revert SDTransferFailed();
        }
```
*Instances(1)*
```
File: contracts/Auction.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L55
```

### [Gas-13] `address(this)` can be stored in `state variable` that will ultimately cost less, than every time calculating this, specifically when it used multiple times in a `contract` 

*Instances(14)*
```
File: contracts/Auction.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L55
```
```
file: contracts/ValidatorWithdrawalVault.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L31
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L32
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L33
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L34
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L55
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L56
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L57
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L94
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L95
```
```
File: contracts/NodeELRewardVault.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L25
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L26
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L27
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L28
```

### [Gas-14] Writing to State variable could be more gas efficient with assembly code

*Instances(6)*
```
File: contracts/Auction.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L144-L149
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L151-L155
```
```
File: contracts/VaultProxy.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L59
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L70
```
```
File: StaderConfig.sol
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L476-L480
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L481-L485
```

