| [Low-1](#Low-1) | `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 1 |
| [Low-2](#Low-2) | Initializers could be front-run | 49 |
| [Low-3](#Low-3) | Unsafe ERC20 operation(s) | 7 |
| [Low-4](#Low-4) | The `owner` is a single point of failure and a centralization risk | 2 |
| [Low-5](#Low-5) | Loss of precision | 11 |
| [Low-6](#Low-6) | Void constructor | 3 |
| [Low-7](#Low-7) | `Keccak` Constant values should used to `immutable` rather than `constant` | 47 |
| [Low-8](#Low-8) | Gas griefing/theft is possible on unsafe external call | 7 |
| [Low-9](#Low-9) | Use `safetransfer` Instead Of `transfer` | 4 |
| [Low-10](#Low-10) | Use `_safeMint` instead of `_mint` | 1 |
| [Low-11](#Low-11) | Division before multiplication causing significant loss of precision | 1 |



### **[Low-1]** `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`

Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). Unless there is a compelling reason, `abi.encode` should be preferred. If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/SocializingPool.sol
174:        bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));
```
</details>
*Instances (1)*

### **[Low-2]** Initializers could be front-run

Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/Auction.sol
29:    function initialize(address _admin, address _staderConfig) external initializer {
33:        __AccessControl_init();
34:        __Pausable_init();
35:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/ETHx.sol
29:    function initialize(address _admin, address _staderConfig) public initializer {
33:        __ERC20_init('ETHx', 'ETHx');
34:        __Pausable_init();
35:        __AccessControl_init();
```
```solidity
File:2023-06-stader/contracts/OperatorRewardsCollector.sol
27:    function initialize(address _admin, address _staderConfig) external initializer {
31:        __AccessControl_init();
32:        __Pausable_init();
```
```solidity
File:2023-06-stader/contracts/Penalty.sol
31:    function initialize(
35:    ) external initializer {
40:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/PermissionedNodeRegistry.sol
66:    function initialize(address _admin, address _staderConfig) public initializer {
70:        __Pausable_init();
71:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/PermissionedPool.sol
40:    function initialize(address _admin, address _staderConfig) public initializer {
44:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/PermissionlessNodeRegistry.sol
66:    function initialize(address _admin, address _staderConfig) public initializer {
70:        __Pausable_init();
71:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/PermissionlessPool.sol
38:    function initialize(address _admin, address _staderConfig) public initializer {
42:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/PoolSelector.sol
36:    function initialize(address _admin, address _staderConfig) external initializer {
```
```solidity
File:2023-06-stader/contracts/PoolUtils.sol
25:    function initialize(address _admin, address _staderConfig) public initializer {
```
```solidity
File:2023-06-stader/contracts/SDCollateral.sol
26:    function initialize(address _admin, address _staderConfig) public initializer {
30:        __AccessControl_init();
31:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/SocializingPool.sol
39:    function initialize(address _admin, address _staderConfig) public initializer {
43:        __AccessControl_init();
44:        __Pausable_init();
45:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/StaderConfig.sol
85:    function initialize(address _admin, address _ethDepositContract) external initializer {
88:        __AccessControl_init();
```
```solidity
File:2023-06-stader/contracts/StaderInsuranceFund.sol
26:    function initialize(address _admin, address _staderConfig) public initializer {
30:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/StaderOracle.sol
62:    function initialize(address _admin, address _staderConfig) public initializer {
66:        __AccessControl_init();
67:        __Pausable_init();
68:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/StaderStakePoolsManager.sol
50:    function initialize(address _admin, address _staderConfig) public initializer {
53:        __AccessControl_init();
54:        __Pausable_init();
55:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/UserWithdrawalManager.sol
54:    function initialize(address _admin, address _staderConfig) public initializer {
58:        __Pausable_init();
59:        __ReentrancyGuard_init();
```
```solidity
File:2023-06-stader/contracts/factory/VaultFactory.sol
23:    function initialize(address _admin, address _staderConfig) external initializer {
```
</details>
*Instances (49)*

### **[Low-3]** Unsafe ERC20 operation(s)

ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard. It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/Auction.sol
55:        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
87:        if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {
114:        if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {
```
```solidity
File:2023-06-stader/contracts/SDCollateral.sol
47:        if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {
68:        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
106:        IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);
```
```solidity
File:2023-06-stader/contracts/SocializingPool.sol
129:            if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {
```
</details>
*Instances (7)*

### **[Low-4]** The `owner` is a single point of failure and a centralization risk

Having a single EOA as the only owner of contracts is a large centralization risk and a single point of failure. A single private key may be taken in a hack, or the sole holder of the key may become unable to retrieve the key when necessary. Consider changing to a multi-signature setup, or having a role-based authorization model.
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/VaultProxy.sol
57:    function updateStaderConfig(address _staderConfig) external override onlyOwner {
68:    function updateOwner(address _owner) external override onlyOwner {
```
</details>
*Instances (2)*

### **[Low-5]** Loss of precision

Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator.
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/PermissionedNodeRegistry.sol
201:        uint256 validatorPerOperator = _numValidators / totalActiveOperatorCount;
```
```solidity
File:2023-06-stader/contracts/PermissionlessPool.sol
128:        uint256 requiredValidators = msg.value / (staderConfig.getFullDepositSize() - DEPOSIT_NODE_BOND);
```
```solidity
File:2023-06-stader/contracts/PoolSelector.sol
61:        uint256 poolTotalTarget = (poolWeights[_poolId] * totalValidatorsRequired) / POOL_WEIGHTS_SUM;
93:            uint256 remainingValidatorsToDeposit = ethToDeposit / poolDepositSize;
```
```solidity
File:2023-06-stader/contracts/PoolUtils.sol
263:        protocolShare = (protocolFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();
265:        operatorShare = (_totalRewards * collateralETH) / TOTAL_STAKED_ETH;
266:        operatorShare += (operatorFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();
```
```solidity
File:2023-06-stader/contracts/StaderOracle.sol
255:        if ((submissionCount == trustedNodesCount / 2 + 1)) {
290:        if ((submissionCount == (2 * trustedNodesCount) / 3 + 1)) {
482:        if ((submissionCount == trustedNodesCount / 2 + 1)) {
```
```solidity
File:2023-06-stader/contracts/UserWithdrawalManager.sol
136:            uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);
```
</details>
*Instances (11)*

### **[Low-6]** Void constructor

Detect the call to a constructor that is not implemented.
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/NodeELRewardVault.sol
14:    constructor() {}
```
```solidity
File:2023-06-stader/contracts/ValidatorWithdrawalVault.sol
23:    constructor() {}
```
```solidity
File:2023-06-stader/contracts/VaultProxy.sol
17:    constructor() {}
```
</details>
*Instances (3)*


### **[Low-7]** `Keccak` Constant values should used to `immutable` rather than `constant`

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/ETHx.sol
21:    bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');
22:    bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');
```
```solidity
File:2023-06-stader/contracts/StaderConfig.sol
12:    bytes32 public constant ETH_PER_NODE = keccak256('ETH_PER_NODE');
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
31:    bytes32 public constant MAX_WITHDRAW_AMOUNT = keccak256('MAX_WITHDRAW_AMOUNT');
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
47:    bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');
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
```solidity
File:2023-06-stader/contracts/StaderOracle.sol
50:    bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency
51:    bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key
52:    bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key
53:    bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key
54:    bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key
```
```solidity
File:2023-06-stader/contracts/factory/VaultFactory.sol
16:    bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');
```
</details>
*Instances (47)*

### **[Low-8]** Gas griefing/theft is possible on unsafe external call

`return` data `(bool success,)` has to be stored due to EVM architecture, if in a usage like below, ‘out’ and ‘outsize’ values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided
<details><summary><i>instances of this issue:</i></summary>


```solidity
File:2023-06-stader/contracts/Auction.sol
131:        (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');
```
```solidity
File:2023-06-stader/contracts/SocializingPool.sol
91:        (bool success, ) = payable(staderConfig.getStaderTreasury()).call{value: _rewardsData.protocolETHRewards}('');
121:            (success, ) = payable(operatorRewardsAddr).call{value: totalAmountETH}('');
```
```solidity
File:2023-06-stader/contracts/StaderInsuranceFund.sol
48:        (bool success, ) = payable(msg.sender).call{value: _amount}('');
```
```solidity
File:2023-06-stader/contracts/StaderStakePoolsManager.sol
102:        (bool success, ) = payable(staderConfig.getUserWithdrawManager()).call{value: _amount}('');
```
```solidity
File:2023-06-stader/contracts/UserWithdrawalManager.sol
229:        (bool success, ) = _recipient.call{value: _amount}('');
```
```solidity
File:2023-06-stader/contracts/library/UtilLib.sol
168:        (bool success, ) = payable(_receiver).call{value: _amount}('');
```
</details>
*Instances (7)*

### **[Low-9]** Use `safetransfer` Instead Of `transfer`


```solidity
File:2023-06-stader/contracts/Auction.sol
87:        if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {
114:        if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {
```
```solidity
File:2023-06-stader/contracts/SDCollateral.sol
68:        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
```
```solidity
File:2023-06-stader/contracts/SocializingPool.sol
129:            if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {
```

*Instances (4)*

### **[Low-10]** Use `_safeMint` instead of `_mint`


```solidity
File:2023-06-stader/contracts/ETHx.sol
48:        _mint(to, amount);
```

*Instances (1)*

### **[Low-11]** Division before multiplication causing significant loss of precision

First divides and then multiplies again, there is a significant loss of precision

```solidity
File:2023-06-stader/contracts/StaderOracle.sol
603:        return (block.number / updateFrequency) * updateFrequency;
```

*Instances (1)*