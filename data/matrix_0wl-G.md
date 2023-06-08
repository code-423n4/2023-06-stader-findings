## Gas Optimizations

|        | Issue                                                                                                                                               |
| ------ | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | Use `selfbalance()` instead of `address(this).balance`                                                                                              |
| GAS-2  | `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`                                                                                          |
| GAS-3  | Use assembly to check for `address(0)`                                                                                                              |
| GAS-4  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                                                                         |
| GAS-5  | USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS                                                            |
| GAS-6  | Use custom errors rather than revert()/require() strings to save gas                                                                                |
| GAS-7  | USE FUNCTION INSTEAD OF MODIFIERS                                                                                                                   |
| GAS-8  | INSTEAD OF CALCULATING A STATEVAR WITH KECCAK256() EVERY TIME THE CONTRACT IS MADE PRE CALCULATE THEM BEFORE AND ONLY GIVE THE RESULT TO A CONSTANT |
| GAS-9  | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT                                                            |
| GAS-10 | USING PRIVATE RATHER THAN PUBLIC FOR CONSTANTS, SAVES GAS                                                                                           |
| GAS-11 | USING SOLIDITY VERSION 0.8.17 WILL PROVIDE AN OVERALL GAS OPTIMIZATION                                                                              |
| GAS-12 | Use shift Right/Left instead of division/multiplication if possible                                                                                 |
| GAS-13 | Short-circuiting                                                                                                                                    |
| GAS-14 | Use bitmaps to save gas                                                                                                                             |
| GAS-15 | USE BYTES32 INSTEAD OF STRING                                                                                                                       |
| GAS-16 | `internal` functions not called by the contract should be removed                                                                                   |

### [GAS-1] Use `selfbalance()` instead of `address(this).balance`

#### Description:

Use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.Use assembly when getting a contract's balance of ETH.

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas.
Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

#### **Proof Of Concept**

```solidity
File: contracts/NodeELRewardVault.sol

28:         uint256 totalRewards = address(this).balance;

```

```solidity
File: contracts/PermissionedPool.sol

174:         if (preDepositValidatorCount != 0 || address(this).balance == 0) {

178:             value: address(this).balance

```

```solidity
File: contracts/SocializingPool.sol

69:             address(this).balance - totalOperatorETHRewardsRemaining

```

```solidity
File: contracts/StaderInsuranceFund.sol

43:         if (address(this).balance < _amount || _amount == 0) {

62:         if (address(this).balance < _amount) {

```

```solidity
File: contracts/StaderStakePoolsManager.sol

189:             address(this).balance,

221:             address(this).balance,

```

```solidity
File: contracts/UserWithdrawalManager.sol

224:         if (address(this).balance < _amount) {

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

34:         uint256 totalRewards = address(this).balance;

99:         uint256 contractBalance = address(this).balance;

```

### [GAS-2] `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`

#### Description:

Use `abi.encodePacked()` where possible to save gas

#### **Proof Of Concept**

```solidity
File: contracts/StaderOracle.sol

135:             abi.encode(_exchangeRate.reportingBlockNumber, _exchangeRate.totalETHBalance, _exchangeRate.totalETHXSupply)

282:         bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _sdPriceData.reportingBlockNumber));

283:         bytes32 submissionCountKey = keccak256(abi.encode(_sdPriceData.reportingBlockNumber));

407:         bytes memory encodedPubkeys = abi.encode(_withdrawnValidators.sortedPubkeys);

418:             abi.encode(_withdrawnValidators.reportingBlockNumber, _withdrawnValidators.nodeRegistry, encodedPubkeys)

466:         bytes memory encodedPubkeys = abi.encode(_mapd.sortedPubkeys);

469:         bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _mapd.index, encodedPubkeys));

470:         bytes32 submissionCountKey = keccak256(abi.encode(_mapd.index, encodedPubkeys));

```

```solidity
File: contracts/factory/VaultFactory.sol

40:         bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));

54:         bytes32 salt = sha256(abi.encode(_poolId, _operatorId));

67:         bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));

77:         bytes32 salt = sha256(abi.encode(_poolId, _operatorId));

```

### [GAS-3] Use assembly to check for `address(0)`

#### **Proof Of Concept**

```solidity
File: contracts/UserWithdrawalManager.sol

96:         if (_owner == address(0)) revert ZeroAddressReceived();

```

```solidity
File: contracts/library/UtilLib.sol

22:         if (_address == address(0)) revert ZeroAddress();

```

### [GAS-4] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

55:         if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {

87:         if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {

114:         if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {

```

```solidity
File: contracts/SDCollateral.sol

68:         if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {

```

```solidity
File: contracts/SocializingPool.sol

129:             if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {

```

### [GAS-5] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS

#### Description:

Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

#### **Proof Of Concept**

```solidity
File: contracts/SDCollateral.sol

124:         string memory _units

```

### [GAS-6] Use custom errors rather than revert()/require() strings to save gas

#### Description:

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they’re hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas.[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.
[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: contracts/VaultProxy.sol

47:             revert(string(data));

```

### [GAS-7] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: contracts/PoolUtils.sol

295:     modifier onlyExistingPoolId(uint8 _poolId) {

```

```solidity
File: contracts/StaderOracle.sol

697:     modifier checkERInspectionMode() {

704:     modifier trustedNodeOnly() {

711:     modifier checkMinTrustedNodes() {

```

```solidity
File: contracts/VaultProxy.sol

75:     modifier onlyOwner() {

```

### [GAS-8] INSTEAD OF CALCULATING A STATEVAR WITH KECCAK256() EVERY TIME THE CONTRACT IS MADE PRE CALCULATE THEM BEFORE AND ONLY GIVE THE RESULT TO A CONSTANT

#### **Proof Of Concept**

```solidity
File: contracts/SocializingPool.sol

174:         bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));

```

```solidity
File: contracts/StaderOracle.sol

126:         bytes32 nodeSubmissionKey = keccak256(

134:         bytes32 submissionCountKey = keccak256(

220:         bytes32 nodeSubmissionKey = keccak256(

232:         bytes32 submissionCountKey = keccak256(

282:         bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _sdPriceData.reportingBlockNumber));

283:         bytes32 submissionCountKey = keccak256(abi.encode(_sdPriceData.reportingBlockNumber));

333:         bytes32 nodeSubmissionKey = keccak256(

345:         bytes32 submissionCountKey = keccak256(

409:         bytes32 nodeSubmissionKey = keccak256(

417:         bytes32 submissionCountKey = keccak256(

469:         bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _mapd.index, encodedPubkeys));

470:         bytes32 submissionCountKey = keccak256(abi.encode(_mapd.index, encodedPubkeys));

```

### [GAS-9] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

#### **Proof Of Concept**

```solidity
File: contracts/ETHx.sol

21:     bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');

22:     bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');

```

```solidity
File: contracts/StaderConfig.sol

12:     bytes32 public constant ETH_PER_NODE = keccak256('ETH_PER_NODE');

14:     bytes32 public constant PRE_DEPOSIT_SIZE = keccak256('PRE_DEPOSIT_SIZE');

16:     bytes32 public constant FULL_DEPOSIT_SIZE = keccak256('FULL_DEPOSIT_SIZE');

18:     bytes32 public constant DECIMALS = keccak256('DECIMALS');

20:     bytes32 public constant TOTAL_FEE = keccak256('TOTAL_FEE');

22:     bytes32 public constant OPERATOR_MAX_NAME_LENGTH = keccak256('OPERATOR_MAX_NAME_LENGTH');

24:     bytes32 public constant SOCIALIZING_POOL_CYCLE_DURATION = keccak256('SOCIALIZING_POOL_CYCLE_DURATION');

27:     bytes32 public constant REWARD_THRESHOLD = keccak256('REWARD_THRESHOLD');

28:     bytes32 public constant MIN_DEPOSIT_AMOUNT = keccak256('MIN_DEPOSIT_AMOUNT');

29:     bytes32 public constant MAX_DEPOSIT_AMOUNT = keccak256('MAX_DEPOSIT_AMOUNT');

30:     bytes32 public constant MIN_WITHDRAW_AMOUNT = keccak256('MIN_WITHDRAW_AMOUNT');

31:     bytes32 public constant MAX_WITHDRAW_AMOUNT = keccak256('MAX_WITHDRAW_AMOUNT');

35:     bytes32 public constant WITHDRAWN_KEYS_BATCH_SIZE = keccak256('WITHDRAWN_KEYS_BATCH_SIZE');

37:     bytes32 public constant ADMIN = keccak256('ADMIN');

38:     bytes32 public constant STADER_TREASURY = keccak256('STADER_TREASURY');

40:     bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');

41:     bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');

42:     bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');

43:     bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');

44:     bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');

45:     bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');

46:     bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');

47:     bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');

48:     bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');

49:     bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');

50:     bytes32 public constant override ETH_DEPOSIT_CONTRACT = keccak256('ETH_DEPOSIT_CONTRACT');

51:     bytes32 public constant override PERMISSIONLESS_POOL = keccak256('PERMISSIONLESS_POOL');

52:     bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');

53:     bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');

54:     bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');

55:     bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');

56:     bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');

57:     bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');

64:     bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');

65:     bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');

68:     bytes32 public constant override MANAGER = keccak256('MANAGER');

69:     bytes32 public constant override OPERATOR = keccak256('OPERATOR');

71:     bytes32 public constant SD = keccak256('SD');

72:     bytes32 public constant ETHx = keccak256('ETHx');

```

```solidity
File: contracts/StaderOracle.sol

50:     bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency

51:     bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key

52:     bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key

53:     bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key

54:     bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key

```

```solidity
File: contracts/factory/VaultFactory.sol

16:     bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');

```

### [GAS-10] USING PRIVATE RATHER THAN PUBLIC FOR CONSTANTS, SAVES GAS

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple. There are four instances of public constants.

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

22:     uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours

```

```solidity
File: contracts/ETHx.sol

21:     bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');

22:     bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

31:     uint8 public constant override POOL_ID = 2;

```

```solidity
File: contracts/PermissionedPool.sol

33:     uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

30:     uint8 public constant override POOL_ID = 1;

42:     uint256 public constant override FRONT_RUN_PENALTY = 3 ether;

43:     uint256 public constant COLLATERAL_ETH = 4 ether;

```

```solidity
File: contracts/PermissionlessPool.sol

23:     uint256 public constant DEPOSIT_NODE_BOND = 3 ether;

31:     uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

```

```solidity
File: contracts/PoolSelector.sol

21:     uint256 public constant POOL_WEIGHTS_SUM = 10000;

```

```solidity
File: contracts/StaderConfig.sol

12:     bytes32 public constant ETH_PER_NODE = keccak256('ETH_PER_NODE');

14:     bytes32 public constant PRE_DEPOSIT_SIZE = keccak256('PRE_DEPOSIT_SIZE');

16:     bytes32 public constant FULL_DEPOSIT_SIZE = keccak256('FULL_DEPOSIT_SIZE');

18:     bytes32 public constant DECIMALS = keccak256('DECIMALS');

20:     bytes32 public constant TOTAL_FEE = keccak256('TOTAL_FEE');

22:     bytes32 public constant OPERATOR_MAX_NAME_LENGTH = keccak256('OPERATOR_MAX_NAME_LENGTH');

24:     bytes32 public constant SOCIALIZING_POOL_CYCLE_DURATION = keccak256('SOCIALIZING_POOL_CYCLE_DURATION');

25:     bytes32 public constant SOCIALIZING_POOL_OPT_IN_COOLING_PERIOD =

27:     bytes32 public constant REWARD_THRESHOLD = keccak256('REWARD_THRESHOLD');

28:     bytes32 public constant MIN_DEPOSIT_AMOUNT = keccak256('MIN_DEPOSIT_AMOUNT');

29:     bytes32 public constant MAX_DEPOSIT_AMOUNT = keccak256('MAX_DEPOSIT_AMOUNT');

30:     bytes32 public constant MIN_WITHDRAW_AMOUNT = keccak256('MIN_WITHDRAW_AMOUNT');

31:     bytes32 public constant MAX_WITHDRAW_AMOUNT = keccak256('MAX_WITHDRAW_AMOUNT');

33:     bytes32 public constant MIN_BLOCK_DELAY_TO_FINALIZE_WITHDRAW_REQUEST =

35:     bytes32 public constant WITHDRAWN_KEYS_BATCH_SIZE = keccak256('WITHDRAWN_KEYS_BATCH_SIZE');

37:     bytes32 public constant ADMIN = keccak256('ADMIN');

38:     bytes32 public constant STADER_TREASURY = keccak256('STADER_TREASURY');

40:     bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');

41:     bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');

42:     bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');

43:     bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');

44:     bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');

45:     bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');

46:     bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');

47:     bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');

48:     bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');

49:     bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');

50:     bytes32 public constant override ETH_DEPOSIT_CONTRACT = keccak256('ETH_DEPOSIT_CONTRACT');

51:     bytes32 public constant override PERMISSIONLESS_POOL = keccak256('PERMISSIONLESS_POOL');

52:     bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');

53:     bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');

54:     bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');

55:     bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');

56:     bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');

57:     bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');

58:     bytes32 public constant override NODE_EL_REWARD_VAULT_IMPLEMENTATION =

60:     bytes32 public constant override VALIDATOR_WITHDRAWAL_VAULT_IMPLEMENTATION =

64:     bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');

65:     bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');

68:     bytes32 public constant override MANAGER = keccak256('MANAGER');

69:     bytes32 public constant override OPERATOR = keccak256('OPERATOR');

71:     bytes32 public constant SD = keccak256('SD');

72:     bytes32 public constant ETHx = keccak256('ETHx');

```

```solidity
File: contracts/StaderOracle.sol

26:     uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days

27:     uint256 public constant ER_CHANGE_MAX_BPS = 10000;

29:     uint256 public constant MIN_TRUSTED_NODES = 5;

50:     bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency

51:     bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key

52:     bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key

53:     bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key

54:     bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key

```

```solidity
File: contracts/factory/VaultFactory.sol

16:     bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');

```

### [GAS-11] USING SOLIDITY VERSION 0.8.17 WILL PROVIDE AN OVERALL GAS OPTIMIZATION

#### **Proof Of Concept**

```solidity
File: contracts/Auction.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/ETHx.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/NodeELRewardVault.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/OperatorRewardsCollector.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/Penalty.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionedNodeRegistry.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionedPool.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PermissionlessPool.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PoolSelector.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/PoolUtils.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/SDCollateral.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/SocializingPool.sol

3: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderConfig.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderInsuranceFund.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderOracle.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/StaderStakePoolsManager.sol

3: pragma solidity 0.8.16;

```

```solidity
File: contracts/UserWithdrawalManager.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/ValidatorWithdrawalVault.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/VaultProxy.sol

2: pragma solidity ^0.8.16;

```

```solidity
File: contracts/factory/VaultFactory.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/library/UtilLib.sol

2: pragma solidity 0.8.16;

```

```solidity
File: contracts/library/ValidatorStatus.sol

2: pragma solidity 0.8.16;

```

### [GAS-12] Use shift Right/Left instead of division/multiplication if possible

#### **Proof Of Concept**

```solidity
File: contracts/StaderOracle.sol

148:             submissionCount == trustedNodesCount / 2 + 1 &&

255:         if ((submissionCount == trustedNodesCount / 2 + 1)) {

314:         return (dataArray[(len - 1) / 2] + dataArray[len / 2]) / 2;

372:             submissionCount == trustedNodesCount / 2 + 1 &&

432:             submissionCount == trustedNodesCount / 2 + 1 &&

482:         if ((submissionCount == trustedNodesCount / 2 + 1)) {

```

### [GAS-13] Short-circuiting

#### Description:

Short-circuiting is a strategy we can make use of when an operation makes use of either || or &&. This pattern works by ordering the lower-cost operation first so that the higher-cost operation may be skipped (short-circuited) if the first operation evaluates to true.
[Source](https://betterprogramming.pub/how-to-write-smart-contracts-that-optimize-gas-spent-on-ethereum-30b5e9c5db85)

#### **Proof Of Concept**

```solidity
File: contracts/StaderInsuranceFund.sol

43:         if (address(this).balance < _amount || _amount == 0) {

```

### [GAS-14] Use bitmaps to save gas

#### Description:

Use mapping(uint256 => uint256) instead of mapping(uint256 => bool)

#### **Proof Of Concept**

```solidity
File: contracts/SocializingPool.sol

29:     mapping(address => mapping(uint256 => bool)) public override claimedRewards;

30:     mapping(uint256 => bool) public handledRewards;

```

#### Recommended Mitigation Steps:

Use mapping(uint256 => uint256) instead of mapping(uint256 => bool)

### [GAS-15] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: contracts/PermissionedNodeRegistry.sol

106:     function onboardNodeOperator(string calldata _operatorName, address payable _operatorRewardAddress)

401:     function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {

661:     function onboardOperator(string calldata _operatorName, address payable _operatorRewardAddress) internal {

```

```solidity
File: contracts/PermissionlessNodeRegistry.sol

91:         string calldata _operatorName,

366:     function updateOperatorDetails(string calldata _operatorName, address payable _rewardAddress) external override {

600:         string calldata _operatorName,

```

```solidity
File: contracts/PoolUtils.sol

215:     function onlyValidName(string calldata _name) external view {

```

```solidity
File: contracts/SDCollateral.sol

124:         string memory _units

```

```solidity
File: contracts/VaultProxy.sol

47:             revert(string(data));

```

### [GAS-16] `internal` functions not called by the contract should be removed

#### Description:

If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword.

#### **Proof Of Concept**

```solidity
File: contracts/library/UtilLib.sol

21:     function checkNonZeroAddress(address _address) internal pure {

26:     function onlyManagerRole(address _addr, IStaderConfig _staderConfig) internal view {

32:     function onlyOperatorRole(address _addr, IStaderConfig _staderConfig) internal view {

39:     function onlyStaderContract(

49:     function getPubkeyForValidSender(

65:     function getOperatorForValidSender(

82:     function onlyValidatorWithdrawVault(

95:     function getOperatorAddressByValidatorId(

107:     function getOperatorAddressByOperatorId(

118:     function getOperatorRewardAddress(address _operator, IStaderConfig _staderConfig)

134:     function getPubkeyRoot(bytes calldata _pubkey) internal pure returns (bytes32) {

143:     function getValidatorSettleStatus(bytes calldata _pubkey, IStaderConfig _staderConfig)

155:     function computeExchangeRate(

167:     function sendValue(address _receiver, uint256 _amount) internal {

```
