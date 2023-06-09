
## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) |  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 1 |
| [L-2](#L-2) | Empty Function Body - Consider commenting why | 3 |
| [L-3](#L-3) | Initializers could be front-run | 64 |
| [L-4](#L-4) | Unsafe ERC20 operation(s) | 7 |
| [NC-1](#NC-1) | Return values of `approve()` not checked | 1 |
| [NC-2](#NC-2) | Functions not used internally could be marked external | 27 |

### [L-1]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (1)*:
```solidity
File: example/SocializingPool.sol

162:         bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));

```

### [L-2] Empty Function Body - Consider commenting why

*Instances (3)*:
```solidity
File: example/NodeELRewardVault.sol

5:     constructor() {}

```

```solidity
File: example/ValidatorWithdrawalVault.sol

9:     constructor() {}

```

```solidity
File: example/VaultProxy.sol

14:     constructor() {}

```

### [L-3] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (64)*:
```solidity
File: example/Auction.sol

19:     function initialize(address _admin, address _staderConfig) external initializer {

19:     function initialize(address _admin, address _staderConfig) external initializer {

23:         __AccessControl_init();

24:         __Pausable_init();

25:         __ReentrancyGuard_init();

```

```solidity
File: example/ETHx.sol

22:     function initialize(address _admin, address _staderConfig) public initializer {

22:     function initialize(address _admin, address _staderConfig) public initializer {

26:         __ERC20_init('ETHx', 'ETHx');

27:         __Pausable_init();

28:         __AccessControl_init();

```

```solidity
File: example/OperatorRewardsCollector.sol

19:     function initialize(address _admin, address _staderConfig) external initializer {

19:     function initialize(address _admin, address _staderConfig) external initializer {

23:         __AccessControl_init();

24:         __Pausable_init();

```

```solidity
File: example/Penalty.sol

21:     function initialize(

25:     ) external initializer {

30:         __ReentrancyGuard_init();

```

```solidity
File: example/PermissionedNodeRegistry.sol

50:     function initialize(address _admin, address _staderConfig) public initializer {

50:     function initialize(address _admin, address _staderConfig) public initializer {

54:         __Pausable_init();

55:         __ReentrancyGuard_init();

```

```solidity
File: example/PermissionedPool.sol

23:     function initialize(address _admin, address _staderConfig) public initializer {

23:     function initialize(address _admin, address _staderConfig) public initializer {

27:         __ReentrancyGuard_init();

```

```solidity
File: example/PermissionlessNodeRegistry.sol

48:     function initialize(address _admin, address _staderConfig) public initializer {

48:     function initialize(address _admin, address _staderConfig) public initializer {

52:         __Pausable_init();

53:         __ReentrancyGuard_init();

```

```solidity
File: example/PermissionlessPool.sol

24:     function initialize(address _admin, address _staderConfig) public initializer {

24:     function initialize(address _admin, address _staderConfig) public initializer {

28:         __ReentrancyGuard_init();

```

```solidity
File: example/PoolSelector.sol

27:     function initialize(address _admin, address _staderConfig) external initializer {

27:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: example/PoolUtils.sol

17:     function initialize(address _admin, address _staderConfig) public initializer {

17:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: example/SDCollateral.sol

14:     function initialize(address _admin, address _staderConfig) public initializer {

14:     function initialize(address _admin, address _staderConfig) public initializer {

18:         __AccessControl_init();

19:         __ReentrancyGuard_init();

```

```solidity
File: example/SocializingPool.sol

27:     function initialize(address _admin, address _staderConfig) public initializer {

27:     function initialize(address _admin, address _staderConfig) public initializer {

31:         __AccessControl_init();

32:         __Pausable_init();

33:         __ReentrancyGuard_init();

```

```solidity
File: example/StaderConfig.sol

79:     function initialize(address _admin, address _ethDepositContract) external initializer {

79:     function initialize(address _admin, address _ethDepositContract) external initializer {

82:         __AccessControl_init();

```

```solidity
File: example/StaderInsuranceFund.sol

18:     function initialize(address _admin, address _staderConfig) public initializer {

18:     function initialize(address _admin, address _staderConfig) public initializer {

22:         __ReentrancyGuard_init();

```

```solidity
File: example/StaderOracle.sol

49:     function initialize(address _admin, address _staderConfig) public initializer {

49:     function initialize(address _admin, address _staderConfig) public initializer {

53:         __AccessControl_init();

54:         __Pausable_init();

55:         __ReentrancyGuard_init();

```

```solidity
File: example/StaderStakePoolsManager.sol

34:     function initialize(address _admin, address _staderConfig) public initializer {

34:     function initialize(address _admin, address _staderConfig) public initializer {

37:         __AccessControl_init();

38:         __Pausable_init();

39:         __ReentrancyGuard_init();

```

```solidity
File: example/UserWithdrawalManager.sol

40:     function initialize(address _admin, address _staderConfig) public initializer {

40:     function initialize(address _admin, address _staderConfig) public initializer {

44:         __Pausable_init();

45:         __ReentrancyGuard_init();

```

### [L-4] Unsafe ERC20 operation(s)

*Instances (7)*:
```solidity
File: example/Auction.sol

45:         if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {

77:         if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {

104:         if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {

```

```solidity
File: example/SDCollateral.sol

35:         if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {

56:         if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {

94:         IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);

```

```solidity
File: example/SocializingPool.sol

117:             if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {

```

### [NC-1] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (1)*:
```solidity
File: example/SDCollateral.sol

94:         IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);

```

### [NC-2] Functions not used internally could be marked external

*Instances (27)*:
```solidity
File: example/ETHx.sol

22:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: example/PermissionedNodeRegistry.sol

50:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: example/PermissionedPool.sol

23:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: example/PermissionlessNodeRegistry.sol

48:     function initialize(address _admin, address _staderConfig) public initializer {

422:     function getTotalQueuedValidatorCount() public view override returns (uint256) {

```

```solidity
File: example/PermissionlessPool.sol

24:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: example/PoolUtils.sol

17:     function initialize(address _admin, address _staderConfig) public initializer {

128:     function getSocializingPoolAddress(uint8 _poolId)

139:     function getOperatorTotalNonTerminalKeys(

```

```solidity
File: example/SDCollateral.sol

14:     function initialize(address _admin, address _staderConfig) public initializer {

193:     function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {

```

```solidity
File: example/SocializingPool.sol

27:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: example/StaderConfig.sol

498:     function onlyManagerRole(address account) public view override returns (bool) {

502:     function onlyOperatorRole(address account) public view override returns (bool) {

```

```solidity
File: example/StaderInsuranceFund.sol

18:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: example/StaderOracle.sol

49:     function initialize(address _admin, address _staderConfig) public initializer {

558:     function getERReportableBlock() public view override returns (uint256) {

569:     function getSDPriceReportableBlock() public view override returns (uint256) {

573:     function getValidatorStatsReportableBlock() public view override returns (uint256) {

577:     function getWithdrawnValidatorReportableBlock() public view override returns (uint256) {

```

```solidity
File: example/StaderStakePoolsManager.sol

34:     function initialize(address _admin, address _staderConfig) public initializer {

109:     function getExchangeRate() public view override returns (uint256) {

124:     function convertToShares(uint256 _assets) public view override returns (uint256) {

129:     function convertToAssets(uint256 _shares) public view override returns (uint256) {

148:     function previewWithdraw(uint256 _shares) public view override returns (uint256) {

153:     function deposit(address _receiver) public payable override whenNotPaused returns (uint256) {

```

```solidity
File: example/UserWithdrawalManager.sol

40:     function initialize(address _admin, address _staderConfig) public initializer {

```


