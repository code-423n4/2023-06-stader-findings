## Medium Issues

### <a name="M-1"></a>[M-1] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (63)*:
```solidity
File: Auction.sol

138:     function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: ETHx.sol

47:     function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) whenNotPaused {

56:     function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {

73:     function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) {

85:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: OperatorRewardsCollector.sol

56:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: Penalty.sol

91:     function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: PermissionedNodeRegistry.sol

459:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: PermissionedPool.sol

248:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: PermissionlessNodeRegistry.sol

354:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: PermissionlessPool.sol

210:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: PoolSelector.sol

147:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: PoolUtils.sol

59:         onlyRole(DEFAULT_ADMIN_ROLE)

84:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: SDCollateral.sol

110:     function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: SocializingPool.sol

179:     function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: StaderConfig.sol

107:     function updateSocializingPoolCycleDuration(uint256 _socializingPoolCycleDuration) external onlyRole(MANAGER) {

113:         onlyRole(MANAGER)

118:     function updateRewardsThreshold(uint256 _rewardsThreshold) external onlyRole(MANAGER) {

126:     function updateMinDepositAmount(uint256 _minDepositAmount) external onlyRole(MANAGER) {

135:     function updateMaxDepositAmount(uint256 _maxDepositAmount) external onlyRole(MANAGER) {

144:     function updateMinWithdrawAmount(uint256 _minWithdrawAmount) external onlyRole(DEFAULT_ADMIN_ROLE) {

153:     function updateMaxWithdrawAmount(uint256 _maxWithdrawAmount) external onlyRole(DEFAULT_ADMIN_ROLE) {

160:         onlyRole(DEFAULT_ADMIN_ROLE)

170:     function updateWithdrawnKeysBatchSize(uint256 _withdrawnKeysBatchSize) external onlyRole(OPERATOR) {

176:     function updateAdmin(address _admin) external onlyRole(DEFAULT_ADMIN_ROLE) {

185:     function updateStaderTreasury(address _staderTreasury) external onlyRole(MANAGER) {

191:     function updatePoolUtils(address _poolUtils) external onlyRole(DEFAULT_ADMIN_ROLE) {

195:     function updatePoolSelector(address _poolSelector) external onlyRole(DEFAULT_ADMIN_ROLE) {

199:     function updateSDCollateral(address _sdCollateral) external onlyRole(DEFAULT_ADMIN_ROLE) {

203:     function updateOperatorRewardsCollector(address _operatorRewardsCollector) external onlyRole(DEFAULT_ADMIN_ROLE) {

207:     function updateVaultFactory(address _vaultFactory) external onlyRole(DEFAULT_ADMIN_ROLE) {

211:     function updateAuctionContract(address _auctionContract) external onlyRole(DEFAULT_ADMIN_ROLE) {

215:     function updateStaderOracle(address _staderOracle) external onlyRole(DEFAULT_ADMIN_ROLE) {

219:     function updatePenaltyContract(address _penaltyContract) external onlyRole(DEFAULT_ADMIN_ROLE) {

223:     function updatePermissionedPool(address _permissionedPool) external onlyRole(DEFAULT_ADMIN_ROLE) {

227:     function updateStakePoolManager(address _stakePoolManager) external onlyRole(DEFAULT_ADMIN_ROLE) {

231:     function updatePermissionlessPool(address _permissionlessPool) external onlyRole(DEFAULT_ADMIN_ROLE) {

235:     function updateUserWithdrawManager(address _userWithdrawManager) external onlyRole(DEFAULT_ADMIN_ROLE) {

239:     function updateStaderInsuranceFund(address _staderInsuranceFund) external onlyRole(DEFAULT_ADMIN_ROLE) {

243:     function updatePermissionedNodeRegistry(address _permissionedNodeRegistry) external onlyRole(DEFAULT_ADMIN_ROLE) {

249:         onlyRole(DEFAULT_ADMIN_ROLE)

256:         onlyRole(DEFAULT_ADMIN_ROLE)

263:         onlyRole(DEFAULT_ADMIN_ROLE)

268:     function updateNodeELRewardImplementation(address _nodeELRewardVaultImpl) external onlyRole(DEFAULT_ADMIN_ROLE) {

274:         onlyRole(DEFAULT_ADMIN_ROLE)

279:     function updateETHBalancePORFeedProxy(address _ethBalanceProxy) external onlyRole(DEFAULT_ADMIN_ROLE) {

283:     function updateETHXSupplyPORFeedProxy(address _ethXSupplyProxy) external onlyRole(DEFAULT_ADMIN_ROLE) {

287:     function updateStaderToken(address _staderToken) external onlyRole(DEFAULT_ADMIN_ROLE) {

291:     function updateETHxToken(address _ethX) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: StaderInsuranceFund.sol

69:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: StaderOracle.sol

503:     function disableSafeMode() external override onlyRole(DEFAULT_ADMIN_ROLE) {

509:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: StaderStakePoolsManager.sol

116:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

261:     function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: UserWithdrawalManager.sol

84:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

201:     function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: VaultProxy.sol

57:     function updateStaderConfig(address _staderConfig) external override onlyOwner {

68:     function updateOwner(address _owner) external override onlyOwner {

```

```solidity
File: factory/VaultFactory.sol

39:     ) public override onlyRole(NODE_REGISTRY_CONTRACT) returns (address) {

51:         onlyRole(NODE_REGISTRY_CONTRACT)

86:     function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {

93:     function updateVaultProxyAddress(address _vaultProxyImpl) external override onlyRole(DEFAULT_ADMIN_ROLE) {

```


## Low Issues

### [L&#x2011;1] Divide before Multiplication

Unnecessary loss of precision caused by divide before multiplication

*Instances (1)*:
```solidity
File: /contracts/StaderOracle.sol

603:        return (block.number / updateFrequency) * updateFrequency;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L603


### [L&#x2011;2] _safeMint() should be used rather than _mint()

_mint() is discouraged in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin and solmate have versions of this function

*Instances (1)*:
```solidity
File: /contracts/ETHx.sol

48:        _mint(to, amount);

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L48

### <a name="L-3"></a>[L-3]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (1)*:
```solidity
File: SocializingPool.sol

174:         bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));

```

### <a name="L-4"></a>[L-4] Empty Function Body - Consider commenting why

*Instances (3)*:
```solidity
File: NodeELRewardVault.sol

14:     constructor() {}

```

```solidity
File: ValidatorWithdrawalVault.sol

23:     constructor() {}

```

```solidity
File: VaultProxy.sol

17:     constructor() {}

```

### <a name="L-5"></a>[L-5] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (66)*:
```solidity
File: Auction.sol

29:     function initialize(address _admin, address _staderConfig) external initializer {

29:     function initialize(address _admin, address _staderConfig) external initializer {

33:         __AccessControl_init();

34:         __Pausable_init();

35:         __ReentrancyGuard_init();

```

```solidity
File: ETHx.sol

29:     function initialize(address _admin, address _staderConfig) public initializer {

29:     function initialize(address _admin, address _staderConfig) public initializer {

33:         __ERC20_init('ETHx', 'ETHx');

34:         __Pausable_init();

35:         __AccessControl_init();

```

```solidity
File: OperatorRewardsCollector.sol

27:     function initialize(address _admin, address _staderConfig) external initializer {

27:     function initialize(address _admin, address _staderConfig) external initializer {

31:         __AccessControl_init();

32:         __Pausable_init();

```

```solidity
File: Penalty.sol

31:     function initialize(

35:     ) external initializer {

40:         __ReentrancyGuard_init();

```

```solidity
File: PermissionedNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

66:     function initialize(address _admin, address _staderConfig) public initializer {

70:         __Pausable_init();

71:         __ReentrancyGuard_init();

```

```solidity
File: PermissionedPool.sol

40:     function initialize(address _admin, address _staderConfig) public initializer {

40:     function initialize(address _admin, address _staderConfig) public initializer {

44:         __ReentrancyGuard_init();

```

```solidity
File: PermissionlessNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

66:     function initialize(address _admin, address _staderConfig) public initializer {

70:         __Pausable_init();

71:         __ReentrancyGuard_init();

```

```solidity
File: PermissionlessPool.sol

38:     function initialize(address _admin, address _staderConfig) public initializer {

38:     function initialize(address _admin, address _staderConfig) public initializer {

42:         __ReentrancyGuard_init();

```

```solidity
File: PoolSelector.sol

36:     function initialize(address _admin, address _staderConfig) external initializer {

36:     function initialize(address _admin, address _staderConfig) external initializer {

```

```solidity
File: PoolUtils.sol

25:     function initialize(address _admin, address _staderConfig) public initializer {

25:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: SDCollateral.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

26:     function initialize(address _admin, address _staderConfig) public initializer {

30:         __AccessControl_init();

31:         __ReentrancyGuard_init();

```

```solidity
File: SocializingPool.sol

39:     function initialize(address _admin, address _staderConfig) public initializer {

39:     function initialize(address _admin, address _staderConfig) public initializer {

43:         __AccessControl_init();

44:         __Pausable_init();

45:         __ReentrancyGuard_init();

```

```solidity
File: StaderConfig.sol

85:     function initialize(address _admin, address _ethDepositContract) external initializer {

85:     function initialize(address _admin, address _ethDepositContract) external initializer {

88:         __AccessControl_init();

```

```solidity
File: StaderInsuranceFund.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

26:     function initialize(address _admin, address _staderConfig) public initializer {

30:         __ReentrancyGuard_init();

```

```solidity
File: StaderOracle.sol

62:     function initialize(address _admin, address _staderConfig) public initializer {

62:     function initialize(address _admin, address _staderConfig) public initializer {

66:         __AccessControl_init();

67:         __Pausable_init();

68:         __ReentrancyGuard_init();

```

```solidity
File: StaderStakePoolsManager.sol

50:     function initialize(address _admin, address _staderConfig) public initializer {

50:     function initialize(address _admin, address _staderConfig) public initializer {

53:         __AccessControl_init();

54:         __Pausable_init();

55:         __ReentrancyGuard_init();

```

```solidity
File: UserWithdrawalManager.sol

54:     function initialize(address _admin, address _staderConfig) public initializer {

54:     function initialize(address _admin, address _staderConfig) public initializer {

58:         __Pausable_init();

59:         __ReentrancyGuard_init();

```

```solidity
File: factory/VaultFactory.sol

23:     function initialize(address _admin, address _staderConfig) external initializer {

23:     function initialize(address _admin, address _staderConfig) external initializer {

```

### <a name="L-6"></a>[L-6] Unsafe ERC20 operation(s)

*Instances (7)*:
```solidity
File: Auction.sol

55:         if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {

87:         if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {

114:         if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {

```

```solidity
File: SDCollateral.sol

47:         if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {

68:         if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {

106:         IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);

```

```solidity
File: SocializingPool.sol

129:             if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {

```



## Non Critical Issues

### [N&#x2011;1] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

*Instances (47)*:
```solidity
File: /contracts/ETHx.sol

21:    bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');

22:    bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L21

```solidity

14:    bytes32 public constant PRE_DEPOSIT_SIZE = keccak256('PRE_DEPOSIT_SIZE');

16:    bytes32 public constant FULL_DEPOSIT_SIZE = keccak256('FULL_DEPOSIT_SIZE');

18:    bytes32 public constant DECIMALS = keccak256('DECIMALS');

20:    bytes32 public constant TOTAL_FEE = keccak256('TOTAL_FEE');

22:    bytes32 public constant OPERATOR_MAX_NAME_LENGTH = keccak256('OPERATOR_MAX_NAME_LENGTH');

24:    bytes32 public constant SOCIALIZING_POOL_CYCLE_DURATION = keccak256('SOCIALIZING_POOL_CYCLE_DURATION');

27:    bytes32 public constant REWARD_THRESHOLD = keccak256('REWARD_THRESHOLD');

28:    bytes32 public constant MIN_DEPOSIT_AMOUNT = keccak256('MIN_DEPOSIT_AMOUNT');

29:    bytes32 public constant MAX_DEPOSIT_AMOUNT = keccak256('MAX_DEPOSIT_AMOUNT');

30:    bytes32 public constant MIN_WITHDRAW_AMOUNT = keccak256('MIN_WITHDRAW_AMOUNT');

31:    bytes32 public
35:    bytes32 public constant WITHDRAWN_KEYS_BATCH_SIZE = keccak256('WITHDRAWN_KEYS_BATCH_SIZE');

37:    bytes32 public constant ADMIN = keccak256('ADMIN');

38:    bytes32 public constant STADER_TREASURY = keccak256('STADER_TREASURY');

40:    bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');

41:    bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');

42:    bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');

43:    bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');

44:    bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');

45:    bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');

46:    bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');

47:    bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY
48:    bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');

49:    bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');

50:    bytes32 public constant override ETH_DEPOSIT_CONTRACT = keccak256('ETH_DEPOSIT_CONTRACT');

51:    bytes32 public constant override PERMISSIONLESS_POOL = keccak256('PERMISSIONLESS_POOL');

52:    bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');

53:    bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');

54:    bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');

55:    bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');

56:    bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');

57:    bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');

64:    bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');

65:    bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');

68:    bytes32 public constant override MANAGER = keccak256('MANAGER');

69:    bytes32 public constant override OPERATOR = keccak256('OPERATOR');

71:    bytes32 public constant SD = keccak256('SD');

72:    bytes32 public constant ETHx = keccak256('ETHx');

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L12

```solidity
File: /contracts/StaderOracle.sol

50:    bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency

51:    bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key

52:    bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key

53:    bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key

54:    bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L50

```solidity
File: /contracts/factory/VaultFactory.sol

16:    bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L16




### [N&#x2011;2] Lines are too long

Recommended by solidity docs to keep lines to 120 characters or lesser

*Instances (9)*:
```solidity
File: /contracts/PoolSelector.sol

71:     * @dev only stader stake pool manager contract can call, update the `poolIdArrayIndexForExcessDeposit` for next cycle calculation

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L71

```solidity
File: /contracts/StaderOracle.sol

53:    bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key

54:    bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L53

```solidity
File: /contracts/interfaces/IPoolUtils.sol

54:    function getTotalActiveValidatorCount() external view returns (uint256); //returns total active validators across all pools

56:    function getActiveValidatorCountByPool(uint8 _poolId) external view returns (uint256); // returns the total number of active validators in a specific pool

58:    function getQueuedValidatorCountByPool(uint8 _poolId) external view returns (uint256); // returns the total number of queued validators in a specific pool

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/interfaces/IPoolUtils.sol#L54

```solidity
File: /contracts/interfaces/IStaderPoolBase.sol

23:    function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external; // sets the commission fees, protocol and operator

31:    function getTotalActiveValidatorCount() external view returns (uint256); // returns the total number of active validators across all operators

33:    function getTotalQueuedValidatorCount() external view returns (uint256); // returns the total number of queued validators across all operators

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/interfaces/IStaderPoolBase.sol#L23


### [N&#x2011;3] delete keyword can be used instead of setting to 0

It's clearer and reflects the intention of the programmer

*Instances (1)*:
```solidity
File: /contracts/Auction.sol

113:        lotItem.sdAmount = 0;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L113


### [N&#x2011;4] Variable names don't follow the Solidity style guide

 Constant variable should be all caps  each word should use all capital letters, jith underscores separating each word (CONSTANT_CASE)

*Instances (28)*:
```solidity
File: /contracts/PermissionedNodeRegistry.sol

31:    uint8 public constant override POOL_ID = 2;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L31

```solidity
File: /contracts/PermissionlessNodeRegistry.sol

30:    uint8 public constant override POOL_ID = 1;

42:    uint256 public constant override FRONT_RUN_PENALTY = 3 ether;

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L30

```solidity

40:    bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');

41:    bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');

42:    bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');

43:    bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');

44:    bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');

45:    bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');

46:    bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');

47:    bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');

48:    bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');

49:    bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');

50:    bytes32 public constant override ETH_DEPOSIT_

52:    bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');

53:    bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');

54:    bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');

55:    bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');

56:    bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');

57:    bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');

58:    bytes32 public constant override NODE_EL_REWARD_VAULT_IMPLEMENTATION =

60:    bytes32 public constant override VALIDATOR_WITHDRAWAL_VAULT_IMPLEMENTATION =

64:    bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');

65:    bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');

68:    bytes32 public constant override MANAGER = keccak256('MANAGER');

69:    bytes32 public constant override OPERATOR = keccak256('OPERATOR');

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L40

```solidity
File: /contracts/factory/VaultFactory.sol

16:    bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L16


### [N&#x2011;5] Constants in comparisons should appear on the left side

Constants should appear on the left side of comparisons, to avoid accidental assignment

*Instances (47)*:
```solidity
File: /contracts/Auction.sol

64:        if (msg.value == 0) revert InSufficientETH();

98:        if (ethAmount == 0) revert NoBidPlaced();

110:        if (lotItem.sdAmount == 0) revert AlreadyClaimed();

126:        if (withdrawalAmount == 0) revert InSufficientETH();

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L64

```solidity
File: /contracts/NodeELRewardVault.sol

29:        if (totalRewards == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L29

```solidity
File: /contracts/PermissionedNodeRegistry.sol

237:                if (remainingValidatorsToDeposit == 0) {

590:        if (_pageNumber == 0) {

625:        if (_pageNumber == 0) {

631:        if (operatorId == 0) {

697:        if (keyCount == 0 || keyCount > inputKeyCountLimit) {

722:        if (_operatorId == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L237

```solidity
File: /contracts/PermissionedPool.sol

102:            if (validatorToDeposit == 0) {

174:        if (preDepositValidatorCount != 0 || address(this).balance == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L102

```solidity
File: /contracts/PermissionlessNodeRegistry.sol

496:        if (_pageNumber == 0) {

531:        if (_pageNumber == 0) {

537:        if (operatorId == 0) {

565:        if (_pageNumber == 0) {

653:        if (keyCount == 0 || keyCount > inputKeyCountLimit) {

682:        if (_operatorId == 0) {

713:        if (_validatorId == 0 || validatorRegistry[_validatorId].status != ValidatorStatus.INITIALIZED) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L496

```solidity
File: /contracts/PoolUtils.sol

216:        if (bytes(_name).length == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L216

```solidity
File: /contracts/SDCollateral.sol

93:        if (sdSlashed == 0) {

160:        return (getRemainingSDToBond(_operator, _poolId, _numValidator) == 0);

239:        if (bytes(poolThresholdbyPoolId[_poolId].units).length == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L93

```solidity
File: /contracts/SocializingPool.sol

146:            if (_amountSD[i] == 0 && _amountETH[i] == 0) {

146:            if (_amountSD[i] == 0 && _amountETH[i] == 0) {

170:        if (_index == 0 || _index > lastReportedRewardsData.index) {

207:        if (_index == 0) {

212:        if (_index == 1) {

221:        if (rewardsDataMap[_index].reportingBlockNumber == 0) {

222:            if (rewardsDataMap[_index - 1].reportingBlockNumber == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L146

```solidity
File: /contracts/StaderInsuranceFund.sol

43:        if (address(this).balance < _amount || _amount == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L43

```solidity
File: /contracts/StaderOracle.sol

302:        if (sdPrices.length == 1) return;

532:        if (_erChangeLimit == 0 || _erChangeLimit > ER_CHANGE_MAX_BPS) {

560:        if (_updateFrequency == 0) {

600:        if (updateFrequency == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L302

```solidity
File: /contracts/StaderStakePoolsManager.sol

202:        if (selectedPoolCapacity == 0) {

224:        if (availableETHForNewDeposit == 0) {

234:            if (validatorToDeposit == 0) {

274:            (_assets == 0 || supply == 0)

274:            (_assets == 0 || supply == 0)

297:            (supply == 0) ? initialConvertToAssets(_shares, rounding) : _shares.mulDiv(totalAssets(), supply, rounding);

331:                IStaderOracle(staderConfig.getStaderOracle()).getExchangeRate().totalETHXSupply == 0) && (!paused());

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L202

```solidity
File: /contracts/UserWithdrawalManager.sol

174:        if (userRequest.ethExpected == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L174

```solidity
File: /contracts/ValidatorWithdrawalVault.sol

39:        if (totalRewards == 0) {

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L39

```solidity
File: /contracts/library/UtilLib.sol

161:        uint256 newExchangeRate = (totalETHBalance == 0 || totalETHXSupply == 0)

161:        uint256 newExchangeRate = (totalETHBalance == 0 || totalETHXSupply == 0)

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L161

### <a name="NC-6"></a>[NC-6] Missing checks for `address(0)` when assigning values to address state variables

*Instances (3)*:
```solidity
File: Penalty.sol

43:         ratedOracleAddress = _ratedOracleAddress;

86:         ratedOracleAddress = _ratedOracleAddress;

```

```solidity
File: factory/VaultFactory.sol

95:         vaultProxyImplementation = _vaultProxyImpl;

```

### <a name="NC-7"></a>[NC-7] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (1)*:
```solidity
File: SDCollateral.sol

106:         IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);

```

### <a name="NC-8"></a>[NC-8] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (122)*:
```solidity
File: interfaces/IDepositContract.sol

10:     event DepositEvent(bytes pubkey, bytes withdrawal_credentials, bytes amount, bytes signature, bytes index);

```

```solidity
File: interfaces/INodeELRewardVault.sol

12:     event ETHReceived(address indexed sender, uint256 amount);

13:     event Withdrawal(uint256 protocolAmount, uint256 operatorAmount, uint256 userAmount);

14:     event UpdatedStaderConfig(address staderConfig);

```

```solidity
File: interfaces/INodeRegistry.sol

44:     event AddedValidatorKey(address indexed nodeOperator, bytes pubkey, uint256 validatorId);

45:     event ValidatorMarkedAsFrontRunned(bytes pubkey, uint256 validatorId);

46:     event ValidatorWithdrawn(bytes pubkey, uint256 validatorId);

47:     event ValidatorStatusMarkedAsInvalidSignature(bytes pubkey, uint256 validatorId);

48:     event UpdatedValidatorDepositBlock(uint256 validatorId, uint256 depositBlock);

49:     event UpdatedMaxNonTerminalKeyPerOperator(uint64 maxNonTerminalKeyPerOperator);

50:     event UpdatedInputKeyCountLimit(uint256 batchKeyDepositLimit);

51:     event UpdatedStaderConfig(address staderConfig);

52:     event UpdatedOperatorDetails(address indexed nodeOperator, string operatorName, address rewardAddress);

53:     event IncreasedTotalActiveValidatorCount(uint256 totalActiveValidatorCount);

54:     event UpdatedVerifiedKeyBatchSize(uint256 verifiedKeysBatchSize);

55:     event UpdatedWithdrawnKeyBatchSize(uint256 withdrawnKeysBatchSize);

56:     event DecreasedTotalActiveValidatorCount(uint256 totalActiveValidatorCount);

```

```solidity
File: interfaces/IOperatorRewardsCollector.sol

7:     event Claimed(address indexed receiver, uint256 amount);

8:     event DepositedFor(address indexed sender, address indexed receiver, uint256 amount);

```

```solidity
File: interfaces/IPenalty.sol

10:     event UpdatedAdditionalPenaltyAmount(bytes pubkey, uint256 amount);

11:     event UpdatedMEVTheftPenaltyPerStrike(uint256 mevTheftPenalty);

12:     event UpdatedPenaltyOracleAddress(address penaltyOracleAddress);

13:     event UpdatedMissedAttestationPenaltyPerStrike(uint256 missedAttestationPenalty);

14:     event UpdatedValidatorExitPenaltyThreshold(uint256 totalPenaltyThreshold);

15:     event ForceExitValidator(bytes pubkey);

16:     event UpdatedStaderConfig(address staderConfig);

17:     event ValidatorMarkedAsSettled(bytes pubkey);

```

```solidity
File: interfaces/IPermissionedNodeRegistry.sol

15:     event OnboardedOperator(address indexed nodeOperator, address nodeRewardAddress, uint256 operatorId);

16:     event OperatorWhitelisted(address permissionedNO);

17:     event OperatorDeactivated(uint256 operatorID);

18:     event OperatorActivated(uint256 operatorID);

19:     event MaxOperatorIdLimitChanged(uint256 maxOperatorId);

20:     event MarkedValidatorStatusAsPreDeposit(bytes pubkey);

21:     event UpdatedQueuedValidatorIndex(uint256 indexed operatorId, uint256 nextQueuedValidatorIndex);

```

```solidity
File: interfaces/IPermissionlessNodeRegistry.sol

16:     event OnboardedOperator(

22:     event ValidatorMarkedReadyToDeposit(bytes pubkey, uint256 validatorId);

23:     event UpdatedNextQueuedValidatorIndex(uint256 nextQueuedValidatorIndex);

24:     event UpdatedSocializingPoolState(uint256 operatorId, bool optedForSocializingPool, uint256 block);

25:     event TransferredCollateralToPool(uint256 amount);

```

```solidity
File: interfaces/IPoolSelector.sol

12:     event UpdatedPoolWeight(uint8 indexed poolId, uint256 poolWeight);

13:     event UpdatedPoolAllocationMaxSize(uint16 poolAllocationMaxSize);

14:     event UpdatedStaderConfig(address staderConfig);

```

```solidity
File: interfaces/IPoolUtils.sol

20:     event PoolAdded(uint8 indexed poolId, address poolAddress);

21:     event PoolAddressUpdated(uint8 indexed poolId, address poolAddress);

22:     event DeactivatedPool(uint8 indexed poolId, address poolAddress);

23:     event UpdatedStaderConfig(address staderConfig);

24:     event ExitValidator(bytes pubkey);

```

```solidity
File: interfaces/ISocializingPool.sol

43:     event ETHReceived(address indexed sender, uint256 amount);

46:     event OperatorRewardsClaimed(address indexed recipient, uint256 ethRewards, uint256 sdRewards);

47:     event OperatorRewardsUpdated(

54:     event UserETHRewardsTransferred(uint256 ethRewards);

55:     event ProtocolETHRewardsTransferred(uint256 ethRewards);

```

```solidity
File: interfaces/IStaderConfig.sol

13:     event SetConstant(bytes32 key, uint256 amount);

14:     event SetVariable(bytes32 key, uint256 amount);

15:     event SetAccount(bytes32 key, address newAddress);

16:     event SetContract(bytes32 key, address newAddress);

17:     event SetToken(bytes32 key, address newAddress);

```

```solidity
File: interfaces/IStaderInsuranceFund.sol

11:     event ReceivedInsuranceFund(uint256 amount);

12:     event FundWithdrawn(uint256 amount);

13:     event UpdatedStaderConfig(address _staderConfig);

```

```solidity
File: interfaces/IStaderOracle.sol

91:     event ERDataSourceToggled(bool isPORBasedERData);

92:     event UpdatedERChangeLimit(uint256 erChangeLimit);

93:     event ERInspectionModeActivated(bool erInspectionMode, uint256 time);

94:     event ExchangeRateSubmitted(

101:     event ExchangeRateUpdated(uint256 block, uint256 totalEth, uint256 ethxSupply, uint256 time);

104:     event SocializingRewardsMerkleRootSubmitted(

111:     event SocializingRewardsMerkleRootUpdated(uint256 index, bytes32 merkleRoot, uint8 poolId, uint256 block);

112:     event SDPriceSubmitted(address indexed node, uint256 sdPriceInETH, uint256 reportedBlock, uint256 block);

113:     event SDPriceUpdated(uint256 sdPriceInETH, uint256 reportedBlock, uint256 block);

115:     event MissedAttestationPenaltySubmitted(

122:     event MissedAttestationPenaltyUpdated(uint256 index, uint256 block, bytes[] pubkeys);

123:     event UpdateFrequencyUpdated(uint256 updateFrequency);

124:     event ValidatorStatsSubmitted(

135:     event ValidatorStatsUpdated(

145:     event WithdrawnValidatorsSubmitted(

152:     event WithdrawnValidatorsUpdated(uint256 block, address nodeRegistry, bytes[] pubkeys, uint256 time);

155:     event UpdatedStaderConfig(address staderConfig);

```

```solidity
File: interfaces/IStaderPoolBase.sol

13:     event ValidatorPreDepositedOnBeaconChain(bytes pubKey);

14:     event ValidatorDepositedOnBeaconChain(uint256 indexed validatorId, bytes pubKey);

15:     event UpdatedCommissionFees(uint256 protocolFee, uint256 operatorFee);

16:     event ReceivedCollateralETH(uint256 amount);

17:     event UpdatedStaderConfig(address staderConfig);

18:     event ReceivedInsuranceFund(uint256 amount);

19:     event TransferredETHToSSPMForDefectiveKeys(uint256 amount);

```

```solidity
File: interfaces/IStaderStakePoolManager.sol

16:     event UpdatedStaderConfig(address staderConfig);

17:     event Deposited(address indexed caller, address indexed owner, uint256 assets, uint256 shares);

18:     event ExecutionLayerRewardsReceived(uint256 amount);

19:     event AuctionedEthReceived(uint256 amount);

21:     event TransferredETHToUserWithdrawManager(uint256 amount);

22:     event ETHTransferredToPool(uint256 indexed poolId, address poolAddress, uint256 validatorCount);

23:     event WithdrawVaultUserShareReceived(uint256 amount);

24:     event UpdatedExcessETHDepositCoolDown(uint256 excessETHDepositCoolDown);

```

```solidity
File: interfaces/IUserWithdrawalManager.sol

20:     event UpdatedFinalizationBatchLimit(uint256 paginationLimit);

21:     event UpdatedStaderConfig(address staderConfig);

22:     event WithdrawRequestReceived(

30:     event FinalizedWithdrawRequest(uint256 requestId);

31:     event RequestRedeemed(address indexed _sender, address _recipient, uint256 _ethTransferred);

32:     event RecipientAddressUpdated(

38:     event ReceivedETH(uint256 _amount);

```

```solidity
File: interfaces/IValidatorWithdrawalVault.sol

13:     event ETHReceived(address indexed sender, uint256 amount);

14:     event DistributeRewardFailed(uint256 rewardAmount, uint256 rewardThreshold);

15:     event DistributedRewards(uint256 userShare, uint256 operatorShare, uint256 protocolShare);

16:     event SettledFunds(uint256 userShare, uint256 operatorShare, uint256 protocolShare);

17:     event UpdatedStaderConfig(address _staderConfig);

```

```solidity
File: interfaces/IVaultFactory.sol

5:     event WithdrawVaultCreated(address withdrawVault);

6:     event NodeELRewardVaultCreated(address nodeDistributor);

7:     event UpdatedStaderConfig(address staderConfig);

8:     event UpdatedVaultProxyImplementation(address vaultProxyImplementation);

```

```solidity
File: interfaces/SDCollateral/IAuction.sol

23:     event LotCreated(uint256 lotId, uint256 sdAmount, uint256 startBlock, uint256 endBlock, uint256 bidIncrement);

24:     event BidPlaced(uint256 lotId, address indexed bidder, uint256 bid);

25:     event BidWithdrawn(uint256 lotId, address indexed withdrawalAccount, uint256 amount);

26:     event BidCancelled(uint256 lotId);

27:     event SDClaimed(uint256 lotId, address indexed highestBidder, uint256 sdAmount);

28:     event ETHClaimed(uint256 lotId, address indexed sspm, uint256 ethAmount);

29:     event AuctionDurationUpdated(uint256 duration);

30:     event BidIncrementUpdated(uint256 _bidIncrement);

31:     event UnsuccessfulSDAuctionExtracted(uint256 lotId, uint256 sdAmount, address indexed recipient);

```

```solidity
File: interfaces/SDCollateral/ISDCollateral.sol

28:     event SDDeposited(address indexed operator, uint256 sdAmount);

29:     event SDWithdrawn(address indexed operator, uint256 sdAmount);

30:     event SDSlashed(address indexed operator, address indexed auction, uint256 sdSlashed);

31:     event UpdatedPoolThreshold(uint8 poolId, uint256 minThreshold, uint256 withdrawThreshold);

32:     event UpdatedPoolIdForOperator(uint8 poolId, address operator);

```

### <a name="NC-9"></a>[NC-9] Functions not used internally could be marked external

*Instances (31)*:
```solidity
File: ETHx.sol

29:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: PermissionedNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: PermissionedPool.sol

40:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: PermissionlessNodeRegistry.sol

66:     function initialize(address _admin, address _staderConfig) public initializer {

440:     function getTotalQueuedValidatorCount() public view override returns (uint256) {

```

```solidity
File: PermissionlessPool.sol

38:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: PoolUtils.sol

25:     function initialize(address _admin, address _staderConfig) public initializer {

136:     function getSocializingPoolAddress(uint8 _poolId)

147:     function getOperatorTotalNonTerminalKeys(

```

```solidity
File: SDCollateral.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

205:     function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {

```

```solidity
File: SocializingPool.sol

39:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: StaderConfig.sol

504:     function onlyManagerRole(address account) public view override returns (bool) {

508:     function onlyOperatorRole(address account) public view override returns (bool) {

```

```solidity
File: StaderInsuranceFund.sol

26:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: StaderOracle.sol

62:     function initialize(address _admin, address _staderConfig) public initializer {

571:     function getERReportableBlock() public view override returns (uint256) {

582:     function getSDPriceReportableBlock() public view override returns (uint256) {

586:     function getValidatorStatsReportableBlock() public view override returns (uint256) {

590:     function getWithdrawnValidatorReportableBlock() public view override returns (uint256) {

```

```solidity
File: StaderStakePoolsManager.sol

50:     function initialize(address _admin, address _staderConfig) public initializer {

125:     function getExchangeRate() public view override returns (uint256) {

140:     function convertToShares(uint256 _assets) public view override returns (uint256) {

145:     function convertToAssets(uint256 _shares) public view override returns (uint256) {

164:     function previewWithdraw(uint256 _shares) public view override returns (uint256) {

169:     function deposit(address _receiver) public payable override whenNotPaused returns (uint256) {

```

```solidity
File: UserWithdrawalManager.sol

54:     function initialize(address _admin, address _staderConfig) public initializer {

```

```solidity
File: factory/VaultFactory.sol

34:     function deployWithdrawVault(

48:     function deployNodeELRewardVault(uint8 _poolId, uint256 _operatorId)

62:     function computeWithdrawVaultAddress(

71:     function computeNodeELRewardVaultAddress(uint8 _poolId, uint256 _operatorId)

```



