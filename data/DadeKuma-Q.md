# Findings

- [L-01] Missing access control in `createLot`
- [L-02] Additional penalty can't be decreased
- [L-03] Non-onboarded operators can't be removed from the whitelist
- [L-04] Reward events can be emitted by anyone
- [L-05] Missing access control in operator deposit
- [L-06] Users may trigger the vault deposit with zero assets


## [L-01] Missing access control in `createLot`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L48-L60

Anyone can create an auction, but they will get nothing in return as their funds will be lost. This function should be callable only by `SDCollateral`:

```solidity
	function createLot(uint256 _sdAmount) external override whenNotPaused {
	    lots[nextLot].startBlock = block.number;
	    lots[nextLot].endBlock = block.number + duration;
	    lots[nextLot].sdAmount = _sdAmount;

	    LotItem storage lotItem = lots[nextLot];

	    if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
	        revert SDTransferFailed();
	    }
	    emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
	    nextLot++;
	}
```

## [L-02] Additional penalty can't be decreased

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Penalty.sol#L53-L59

The penalty amount can only increase, but not decrease. If this function is called on an intended `_pubkey`, it won't be possible to decrease it again:

```solidity
	function setAdditionalPenaltyAmount(bytes calldata _pubkey, uint256 _amount) external override {
	    UtilLib.onlyManagerRole(msg.sender, staderConfig);
	    bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey);
	    additionalPenaltyAmount[pubkeyRoot] += _amount;

	    emit UpdatedAdditionalPenaltyAmount(_pubkey, _amount);
	}
```


## [L-03] Non-onboarded operators can't be removed from the whitelist

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L88-L97

If an unintended address is added to the operator whitelist by mistake, it's not possible to remove it before they complete the onboarding process:

```solidity
	function whitelistPermissionedNOs(address[] calldata _permissionedNOs) external override {
	    UtilLib.onlyManagerRole(msg.sender, staderConfig);
	    uint256 permissionedNosLength = _permissionedNOs.length;
	    for (uint256 i = 0; i < permissionedNosLength; i++) {
	        address operator = _permissionedNOs[i];
	        UtilLib.checkNonZeroAddress(operator);
	        permissionList[operator] = true;
	        emit OperatorWhitelisted(operator);
	    }
	}
```

## [L-04] Reward events can be emitted by anyone

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderStakePoolsManager.sol#L78-L80

There are multiple `payable` functions that are not protected by access control. Anyone can send 1 wei to trigger these functions, which may be used to trick other components that listen to these events.

For example, this event should be emitted only when the withdraw vault completes a payment, but it's callable by anyone:

```solidity
	// payable function for receiving user share from validator withdraw vault
	function receiveWithdrawVaultUserShare() external payable override {
	    emit WithdrawVaultUserShareReceived(msg.value);
	}
```

## [L-05] Missing access control in operator deposit

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L43-L52

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L58-L73

Anyone can send funds as a deposit, but there isn't an access control to check if they are an operator:

```solidity
	function depositSDAsCollateral(uint256 _sdAmount) external override { 
	    address operator = msg.sender;
	    operatorSDBalance[operator] += _sdAmount;

	    if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {
	        revert SDTransferFailed();
	    }

	    emit SDDeposited(operator, _sdAmount);
	}
```

When these users try to withdraw their funds and they aren't an operator, their transaction will fail, and their funds will be locked:

```solidity
	function withdraw(uint256 _requestedSD) external override {
	    address operator = msg.sender;
	    uint256 opSDBalance = operatorSDBalance[operator];

->	    if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
	        revert InsufficientSDToWithdraw(opSDBalance);
	    }
	    operatorSDBalance[operator] -= _requestedSD;

	    // cannot use safeERC20 as this contract is an upgradeable contract
	    if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
	        revert SDTransferFailed();
	    }

	    emit SDWithdrawn(operator, _requestedSD);
	}
```


## [L-06] Users may trigger the vault deposit with zero assets

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderStakePoolsManager.sol#L169-L177

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderStakePoolsManager.sol#L154-L156

No one can do a deposit unless their assets are greater than the `minDeposit`:

```solidity
	function deposit(address _receiver) public payable override whenNotPaused returns (uint256) {
	    uint256 assets = msg.value;
	    if (assets > maxDeposit() || assets < minDeposit()) {
	        revert InvalidDepositAmount();
	    }
	    uint256 shares = previewDeposit(assets);
	    _deposit(msg.sender, _receiver, assets, shares);
	    return shares;
	}
```

However, when the vault is unhealthy, the `minDeposit` is zero:

```solidity
	function minDeposit() public view override returns (uint256) {
	    return isVaultHealthy() ? staderConfig.getMinDepositAmount() : 0;
	}
```

As such, the previous condition will not revert and the user will be able to call the deposit function without failing.

Consider adding a new check:

```solidity
	if (assets == 0 || assets > maxDeposit() || assets < minDeposit()) {
	    revert InvalidDepositAmount();
	}
```



