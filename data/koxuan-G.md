
## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use `selfbalance()` instead of `address(this).balance` | 12 |
| [GAS-2](#GAS-2) | Use assembly to check for `address(0)` | 2 |
| [GAS-3](#GAS-3) | Using bools for storage incurs overhead | 12 |
| [GAS-4](#GAS-4) | State variables should be cached in stack variables rather than re-reading them from storage | 5 |
| [GAS-5](#GAS-5) | Use calldata instead of memory for function arguments that do not get mutated | 2 |
| [GAS-6](#GAS-6) | For Operations that will not overflow, you could use unchecked | 929 |
| [GAS-7](#GAS-7) | Don't initialize variables with default value | 13 |
| [GAS-8](#GAS-8) | Functions guaranteed to revert when called by normal users can be marked `payable` | 78 |
| [GAS-9](#GAS-9) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 33 |
| [GAS-10](#GAS-10) | Using `private` rather than `public` for constants, saves gas | 63 |
| [GAS-11](#GAS-11) | Use shift Right/Left instead of division/multiplication if possible | 6 |
| [GAS-12](#GAS-12) | Use != 0 instead of > 0 for unsigned integer comparison | 13 |
| [GAS-13](#GAS-13) | `internal` functions not called by the contract should be removed | 14 |
### <a name="GAS-1"></a>[GAS-1] Use `selfbalance()` instead of `address(this).balance`
Use assembly when getting a contract's balance of ETH.

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas.
Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

*Saves 15 gas when checking internal balance, 6 for external*

*Instances (12)*:
```solidity
File: NodeELRewardVault.sol

28:         uint256 totalRewards = address(this).balance;

```

```solidity
File: PermissionedPool.sol

174:         if (preDepositValidatorCount != 0 || address(this).balance == 0) {

178:             value: address(this).balance

```

```solidity
File: PermissionlessNodeRegistry.sol

305:             if (address(feeRecipientAddress).balance > 0) {

```

```solidity
File: SocializingPool.sol

69:             address(this).balance - totalOperatorETHRewardsRemaining

```

```solidity
File: StaderInsuranceFund.sol

43:         if (address(this).balance < _amount || _amount == 0) {

62:         if (address(this).balance < _amount) {

```

```solidity
File: StaderStakePoolsManager.sol

189:             address(this).balance,

221:             address(this).balance,

```

```solidity
File: UserWithdrawalManager.sol

224:         if (address(this).balance < _amount) {

```

```solidity
File: ValidatorWithdrawalVault.sol

34:         uint256 totalRewards = address(this).balance;

99:         uint256 contractBalance = address(this).balance;

```

### <a name="GAS-2"></a>[GAS-2] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (2)*:
```solidity
File: UserWithdrawalManager.sol

96:         if (_owner == address(0)) revert ZeroAddressReceived();

```

```solidity
File: library/UtilLib.sol

22:         if (_address == address(0)) revert ZeroAddress();

```

### <a name="GAS-3"></a>[GAS-3] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (12)*:
```solidity
File: PermissionedNodeRegistry.sol

54:     mapping(address => bool) public override permissionList;

```

```solidity
File: SocializingPool.sol

29:     mapping(address => mapping(uint256 => bool)) public override claimedRewards;

30:     mapping(uint256 => bool) public handledRewards;

```

```solidity
File: StaderOracle.sol

18:     bool public override erInspectionMode;

19:     bool public override isPORFeedBasedERData;

41:     bool public override safeMode;

44:     mapping(address => bool) public override isTrustedNode;

45:     mapping(bytes32 => bool) private nodeSubmissionKeys;

```

```solidity
File: ValidatorWithdrawalVault.sol

19:     bool internal vaultSettleStatus;

```

```solidity
File: VaultProxy.sol

9:     bool public override vaultSettleStatus;

10:     bool public override isValidatorWithdrawalVault;

11:     bool public override isInitialized;

```

### <a name="GAS-4"></a>[GAS-4] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (5)*:
```solidity
File: PermissionedNodeRegistry.sol

173:             validatorIdByPubkey[_pubkey[i]] = nextValidatorId;

202:         uint256[] memory remainingOperatorCapacity = new uint256[](nextOperatorId);

```

```solidity
File: PermissionlessNodeRegistry.sol

159:             validatorIdByPubkey[_pubkey[i]] = nextValidatorId;

```

```solidity
File: StaderOracle.sol

663:         uint256 newExchangeRate = UtilLib.computeExchangeRate(_newTotalETHBalance, _newTotalETHXSupply, staderConfig);

```

```solidity
File: UserWithdrawalManager.sol

108:         emit WithdrawRequestReceived(msg.sender, _owner, nextRequestId, _ethXAmount, assets);

```

### <a name="GAS-5"></a>[GAS-5] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (2)*:
```solidity
File: SDCollateral.sol

124:         string memory _units

```

```solidity
File: interfaces/SDCollateral/ISDCollateral.sol

51:         string memory _units

```

### <a name="GAS-6"></a>[GAS-6] For Operations that will not overflow, you could use unchecked

*Instances (929)*:
```solidity
File: Auction.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import '../contracts/interfaces/SDCollateral/IAuction.sol';

6: import '../contracts/interfaces/SDCollateral/IAuction.sol';

6: import '../contracts/interfaces/SDCollateral/IAuction.sol';

6: import '../contracts/interfaces/SDCollateral/IAuction.sol';

7: import '../contracts/interfaces/IStaderStakePoolManager.sol';

7: import '../contracts/interfaces/IStaderStakePoolManager.sol';

7: import '../contracts/interfaces/IStaderStakePoolManager.sol';

9: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

9: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

9: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

9: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

22:     uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours

22:     uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours

38:         duration = 2 * MIN_AUCTION_DURATION;

50:         lots[nextLot].endBlock = block.number + duration;

59:         nextLot++;

59:         nextLot++;

69:         uint256 totalUserBid = lotItem.bids[msg.sender] + msg.value;

71:         if (totalUserBid < lotItem.highestBidAmount + bidIncrement) revert InSufficientBid();

128:         lotItem.bids[msg.sender] -= withdrawalAmount;

```

```solidity
File: ETHx.sol

3: import './library/UtilLib.sol';

3: import './library/UtilLib.sol';

5: import './interfaces/IStaderConfig.sol';

5: import './interfaces/IStaderConfig.sol';

7: import '@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol';

7: import '@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol';

7: import '@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol';

7: import '@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol';

7: import '@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol';

8: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

8: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

8: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

8: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

```

```solidity
File: NodeELRewardVault.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './interfaces/IPoolUtils.sol';

6: import './interfaces/IPoolUtils.sol';

7: import './interfaces/IVaultProxy.sol';

7: import './interfaces/IVaultProxy.sol';

8: import './interfaces/INodeRegistry.sol';

8: import './interfaces/INodeRegistry.sol';

9: import './interfaces/INodeELRewardVault.sol';

9: import './interfaces/INodeELRewardVault.sol';

10: import './interfaces/IStaderStakePoolManager.sol';

10: import './interfaces/IStaderStakePoolManager.sol';

11: import './interfaces/IOperatorRewardsCollector.sol';

11: import './interfaces/IOperatorRewardsCollector.sol';

```

```solidity
File: OperatorRewardsCollector.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './interfaces/IOperatorRewardsCollector.sol';

6: import './interfaces/IOperatorRewardsCollector.sol';

7: import './interfaces/IStaderConfig.sol';

7: import './interfaces/IStaderConfig.sol';

9: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

41:         balances[_receiver] += msg.value;

49:         balances[operator] -= amount;

```

```solidity
File: Penalty.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './interfaces/IPenalty.sol';

6: import './interfaces/IPenalty.sol';

7: import './interfaces/IRatedV1.sol';

7: import './interfaces/IRatedV1.sol';

8: import './interfaces/IStaderOracle.sol';

8: import './interfaces/IStaderOracle.sol';

9: import './interfaces/IStaderConfig.sol';

9: import './interfaces/IStaderConfig.sol';

11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

56:         additionalPenaltyAmount[pubkeyRoot] += _amount;

111:             uint256 totalPenalty = _mevTheftPenalty + _missedAttestationPenalty + additionalPenaltyAmount[pubkeyRoot];

111:             uint256 totalPenalty = _mevTheftPenalty + _missedAttestationPenalty + additionalPenaltyAmount[pubkeyRoot];

117:                 ++i;

117:                 ++i;

128:         return violatedEpochs.length * mevTheftPenaltyPerStrike;

134:             IStaderOracle(staderConfig.getStaderOracle()).missedAttestationPenalty(_pubkeyRoot) *

```

```solidity
File: PermissionedNodeRegistry.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './library/ValidatorStatus.sol';

6: import './library/ValidatorStatus.sol';

7: import './interfaces/IStaderConfig.sol';

7: import './interfaces/IStaderConfig.sol';

8: import './interfaces/IVaultFactory.sol';

8: import './interfaces/IVaultFactory.sol';

9: import './interfaces/IPoolUtils.sol';

9: import './interfaces/IPoolUtils.sol';

10: import './interfaces/INodeRegistry.sol';

10: import './interfaces/INodeRegistry.sol';

11: import './interfaces/IPermissionedPool.sol';

11: import './interfaces/IPermissionedPool.sol';

12: import './interfaces/IValidatorWithdrawalVault.sol';

12: import './interfaces/IValidatorWithdrawalVault.sol';

13: import './interfaces/SDCollateral/ISDCollateral.sol';

13: import './interfaces/SDCollateral/ISDCollateral.sol';

13: import './interfaces/SDCollateral/ISDCollateral.sol';

14: import './interfaces/IPermissionedNodeRegistry.sol';

14: import './interfaces/IPermissionedNodeRegistry.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

17: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

17: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

17: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

17: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

91:         for (uint256 i = 0; i < permissionedNosLength; i++) {

91:         for (uint256 i = 0; i < permissionedNosLength; i++) {

160:                 operatorTotalKeys + i, //operator totalKeys

160:                 operatorTotalKeys + i, //operator totalKeys

160:                 operatorTotalKeys + i, //operator totalKeys

176:             nextValidatorId++;

176:             nextValidatorId++;

178:                 ++i;

178:                 ++i;

201:         uint256 validatorPerOperator = _numValidators / totalActiveOperatorCount;

212:                 totalValidatorToDeposit += selectedOperatorCapacity[i];

213:                 remainingOperatorCapacity[i] -= selectedOperatorCapacity[i];

215:                     ++i;

215:                     ++i;

223:             uint256 totalOperators = nextOperatorId - 1;

224:             uint256 remainingValidatorsToDeposit = _numValidators - totalValidatorToDeposit;

234:                 selectedOperatorCapacity[i] += newSelectedCapacity;

235:                 remainingValidatorsToDeposit -= newSelectedCapacity;

236:                 i = (i % totalOperators) + 1;

265:             readyToDepositValidatorsLength + frontRunValidatorsLength + invalidSignatureValidatorsLength >

265:             readyToDepositValidatorsLength + frontRunValidatorsLength + invalidSignatureValidatorsLength >

280:                 ++i;

280:                 ++i;

293:                 ++i;

293:                 ++i;

298:         uint256 totalDefectedKeys = frontRunValidatorsLength + invalidSignatureValidatorsLength;

327:                 ++i;

327:                 ++i;

477:         totalActiveValidatorCount += _count;

490:                 totalQueuedValidators += getOperatorQueuedValidatorCount(i);

493:                 ++i;

493:                 ++i;

537:                 totalNonWithdrawnKeyCount++;

537:                 totalNonWithdrawnKeyCount++;

540:                 ++i;

540:                 ++i;

593:         uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;

593:         uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;

593:         uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;

594:         uint256 endIndex = startIndex + _pageSize;

598:         for (uint256 i = startIndex; i < endIndex; i++) {

598:         for (uint256 i = startIndex; i < endIndex; i++) {

601:                 validatorCount++;

601:                 validatorCount++;

628:         uint256 startIndex = (_pageNumber - 1) * _pageSize;

628:         uint256 startIndex = (_pageNumber - 1) * _pageSize;

629:         uint256 endIndex = startIndex + _pageSize;

636:         Validator[] memory validators = new Validator[](endIndex > startIndex ? endIndex - startIndex : 0);

637:         for (uint256 i = startIndex; i < endIndex; i++) {

637:         for (uint256 i = startIndex; i < endIndex; i++) {

639:             validators[i - startIndex] = validatorRegistry[validatorId];

665:         nextOperatorId++;

665:         nextOperatorId++;

666:         totalActiveOperatorCount++;

666:         totalActiveOperatorCount++;

667:         emit OnboardedOperator(msg.sender, _operatorRewardAddress, nextOperatorId - 1);

682:             validatorIdsByOperatorId[_operatorId].length -

702:         if ((totalNonTerminalKeys + keyCount) > maxNonTerminalKeyPerOperator) {

712:                 totalNonTerminalKeys + keyCount

748:         totalActiveValidatorCount -= _count;

764:         totalActiveOperatorCount--;

764:         totalActiveOperatorCount--;

770:         totalActiveOperatorCount++;

770:         totalActiveOperatorCount++;

```

```solidity
File: PermissionedPool.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './library/ValidatorStatus.sol';

6: import './library/ValidatorStatus.sol';

8: import './interfaces/IStaderConfig.sol';

8: import './interfaces/IStaderConfig.sol';

9: import './interfaces/IVaultFactory.sol';

9: import './interfaces/IVaultFactory.sol';

10: import './interfaces/INodeRegistry.sol';

10: import './interfaces/INodeRegistry.sol';

11: import './interfaces/IStaderPoolBase.sol';

11: import './interfaces/IStaderPoolBase.sol';

12: import './interfaces/IDepositContract.sol';

12: import './interfaces/IDepositContract.sol';

13: import './interfaces/IStaderInsuranceFund.sol';

13: import './interfaces/IStaderInsuranceFund.sol';

14: import './interfaces/IStaderStakePoolManager.sol';

14: import './interfaces/IStaderStakePoolManager.sol';

15: import './interfaces/IPermissionedNodeRegistry.sol';

15: import './interfaces/IPermissionedNodeRegistry.sol';

17: import '@openzeppelin/contracts/utils/math/Math.sol';

17: import '@openzeppelin/contracts/utils/math/Math.sol';

17: import '@openzeppelin/contracts/utils/math/Math.sol';

17: import '@openzeppelin/contracts/utils/math/Math.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

73:             _defectiveKeyCount * staderConfig.getPreDepositSize()

77:         uint256 amountToSendToPoolManager = _defectiveKeyCount * staderConfig.getStakedEthPerNode();

91:         uint256 requiredValidators = msg.value / staderConfig.getStakedEthPerNode();

111:                 index < nextQueuedValidatorIndex + validatorToDeposit;

112:                 index++

112:                 index++

119:                 nextQueuedValidatorIndex + validatorToDeposit

122:                 ++i;

122:                 ++i;

164:                 ++i;

164:                 ++i;

238:         if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {

280:         preDepositValidatorCount += _count;

284:         preDepositValidatorCount -= _count;

322:         uint64 value = uint64(_depositAmount / 1 gwei);

```

```solidity
File: PermissionlessNodeRegistry.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './library/ValidatorStatus.sol';

6: import './library/ValidatorStatus.sol';

7: import './interfaces/IStaderConfig.sol';

7: import './interfaces/IStaderConfig.sol';

8: import './interfaces/IVaultFactory.sol';

8: import './interfaces/IVaultFactory.sol';

9: import './interfaces/IPoolUtils.sol';

9: import './interfaces/IPoolUtils.sol';

10: import './interfaces/INodeRegistry.sol';

10: import './interfaces/INodeRegistry.sol';

11: import './interfaces/IPermissionlessPool.sol';

11: import './interfaces/IPermissionlessPool.sol';

12: import './interfaces/INodeELRewardVault.sol';

12: import './interfaces/INodeELRewardVault.sol';

13: import './interfaces/IStaderInsuranceFund.sol';

13: import './interfaces/IStaderInsuranceFund.sol';

14: import './interfaces/IValidatorWithdrawalVault.sol';

14: import './interfaces/IValidatorWithdrawalVault.sol';

15: import './interfaces/SDCollateral/ISDCollateral.sol';

15: import './interfaces/SDCollateral/ISDCollateral.sol';

15: import './interfaces/SDCollateral/ISDCollateral.sol';

16: import './interfaces/IPermissionlessNodeRegistry.sol';

16: import './interfaces/IPermissionlessNodeRegistry.sol';

17: import './interfaces/IOperatorRewardsCollector.sol';

17: import './interfaces/IOperatorRewardsCollector.sol';

19: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

20: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

20: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

20: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

20: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

21: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

21: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

21: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

21: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

145:                 operatorTotalKeys + i, //operator totalKeys

145:                 operatorTotalKeys + i, //operator totalKeys

145:                 operatorTotalKeys + i, //operator totalKeys

162:             nextValidatorId++;

162:             nextValidatorId++;

164:                 ++i;

164:                 ++i;

170:             value: staderConfig.getPreDepositSize() * keyCount

193:             readyToDepositValidatorsLength + frontRunValidatorsLength + invalidSignatureValidatorsLength >

193:             readyToDepositValidatorsLength + frontRunValidatorsLength + invalidSignatureValidatorsLength >

205:                 ++i;

205:                 ++i;

211:                 value: frontRunValidatorsLength * FRONT_RUN_PENALTY

221:                 ++i;

221:                 ++i;

231:                 ++i;

231:                 ++i;

257:                 ++i;

257:                 ++i;

299:             socializingPoolStateChangeBlock[operatorId] + staderConfig.getSocializingPoolOptInCoolingPeriod()

383:         totalActiveValidatorCount += _count;

418:                 totalNonWithdrawnKeyCount++;

418:                 totalNonWithdrawnKeyCount++;

421:                 ++i;

421:                 ++i;

441:         return validatorQueueSize - nextQueuedValidatorIndex;

499:         uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;

499:         uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;

499:         uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;

500:         uint256 endIndex = startIndex + _pageSize;

504:         for (uint256 i = startIndex; i < endIndex; i++) {

504:         for (uint256 i = startIndex; i < endIndex; i++) {

507:                 validatorCount++;

507:                 validatorCount++;

534:         uint256 startIndex = (_pageNumber - 1) * _pageSize;

534:         uint256 startIndex = (_pageNumber - 1) * _pageSize;

535:         uint256 endIndex = startIndex + _pageSize;

542:         Validator[] memory validators = new Validator[](endIndex > startIndex ? endIndex - startIndex : 0);

543:         for (uint256 i = startIndex; i < endIndex; i++) {

543:         for (uint256 i = startIndex; i < endIndex; i++) {

545:             validators[i - startIndex] = validatorRegistry[validatorId];

568:         uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;

568:         uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;

568:         uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;

569:         uint256 endIndex = startIndex + _pageSize;

573:         for (uint256 i = startIndex; i < endIndex; i++) {

573:         for (uint256 i = startIndex; i < endIndex; i++) {

576:                 optOutOperatorCount++;

576:                 optOutOperatorCount++;

612:         nextOperatorId++;

612:         nextOperatorId++;

614:         emit OnboardedOperator(msg.sender, _operatorRewardAddress, nextOperatorId - 1, _optInForSocializingPool);

621:         validatorQueueSize++;

621:         validatorQueueSize++;

638:             value: (COLLATERAL_ETH - staderConfig.getPreDepositSize())

659:         if ((totalNonTerminalKeys + keyCount) > maxNonTerminalKeyPerOperator) {

664:         if (msg.value != keyCount * COLLATERAL_ETH) {

672:                 totalNonTerminalKeys + keyCount

708:         totalActiveValidatorCount -= _count;

```

```solidity
File: PermissionlessPool.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

5: import './library/ValidatorStatus.sol';

5: import './library/ValidatorStatus.sol';

7: import './interfaces/IStaderConfig.sol';

7: import './interfaces/IStaderConfig.sol';

8: import './interfaces/IVaultFactory.sol';

8: import './interfaces/IVaultFactory.sol';

9: import './interfaces/INodeRegistry.sol';

9: import './interfaces/INodeRegistry.sol';

10: import './interfaces/IStaderPoolBase.sol';

10: import './interfaces/IStaderPoolBase.sol';

11: import './interfaces/IDepositContract.sol';

11: import './interfaces/IDepositContract.sol';

12: import './interfaces/IStaderStakePoolManager.sol';

12: import './interfaces/IStaderStakePoolManager.sol';

13: import './interfaces/IPermissionlessNodeRegistry.sol';

13: import './interfaces/IPermissionlessNodeRegistry.sol';

15: import '@openzeppelin/contracts/utils/math/Math.sol';

15: import '@openzeppelin/contracts/utils/math/Math.sol';

15: import '@openzeppelin/contracts/utils/math/Math.sol';

15: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

16: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

16: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

16: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

17: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

17: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

17: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

17: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

68:         if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {

98:                 _operatorTotalKeys + i

117:                 ++i;

117:                 ++i;

128:         uint256 requiredValidators = msg.value / (staderConfig.getFullDepositSize() - DEPOSIT_NODE_BOND);

128:         uint256 requiredValidators = msg.value / (staderConfig.getFullDepositSize() - DEPOSIT_NODE_BOND);

131:             requiredValidators * DEPOSIT_NODE_BOND

137:         for (uint256 i = depositQueueStartIndex; i < requiredValidators + depositQueueStartIndex; ) {

147:                 ++i;

147:                 ++i;

151:             depositQueueStartIndex + requiredValidators

274:         uint64 value = uint64(_depositAmount / 1 gwei);

```

```solidity
File: PoolSelector.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './interfaces/IStaderConfig.sol';

6: import './interfaces/IStaderConfig.sol';

7: import './interfaces/IPoolSelector.sol';

7: import './interfaces/IPoolSelector.sol';

8: import './interfaces/IPoolUtils.sol';

8: import './interfaces/IPoolUtils.sol';

10: import '@openzeppelin/contracts/utils/math/Math.sol';

10: import '@openzeppelin/contracts/utils/math/Math.sol';

10: import '@openzeppelin/contracts/utils/math/Math.sol';

10: import '@openzeppelin/contracts/utils/math/Math.sol';

11: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

11: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

11: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

11: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

12: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

58:         uint256 totalValidatorsRequired = (totalActiveValidatorCount + _newValidatorToRegister);

61:         uint256 poolTotalTarget = (poolWeights[_poolId] * totalValidatorsRequired) / POOL_WEIGHTS_SUM;

61:         uint256 poolTotalTarget = (poolWeights[_poolId] * totalValidatorsRequired) / POOL_WEIGHTS_SUM;

92:             uint256 poolDepositSize = ETH_PER_NODE - poolUtils.getCollateralETH(poolIdArray[i]);

93:             uint256 remainingValidatorsToDeposit = ethToDeposit / poolDepositSize;

95:                 poolAllocationMaxSize - selectedValidatorCount,

98:             selectedValidatorCount += selectedPoolCapacity[i];

99:             ethToDeposit -= selectedPoolCapacity[i] * poolDepositSize;

99:             ethToDeposit -= selectedPoolCapacity[i] * poolDepositSize;

100:             i = (i + 1) % poolCount;

108:                 ++j;

108:                 ++j;

130:         for (uint256 i = 0; i < poolTargetLength; i++) {

130:         for (uint256 i = 0; i < poolTargetLength; i++) {

131:             totalWeight += _poolTargets[i];

```

```solidity
File: PoolUtils.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './interfaces/IPoolUtils.sol';

6: import './interfaces/IPoolUtils.sol';

7: import './interfaces/IStaderPoolBase.sol';

7: import './interfaces/IStaderPoolBase.sol';

8: import './interfaces/IStaderConfig.sol';

8: import './interfaces/IStaderConfig.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

78:                 ++i;

78:                 ++i;

104:         for (uint256 i = 0; i < poolCount; i++) {

104:         for (uint256 i = 0; i < poolCount; i++) {

105:             totalActiveValidatorCount += getActiveValidatorCountByPool(poolIdArray[i]);

168:         for (uint256 i = 0; i < poolCount; i++) {

168:         for (uint256 i = 0; i < poolCount; i++) {

179:         for (uint256 i = 0; i < poolCount; i++) {

179:         for (uint256 i = 0; i < poolCount; i++) {

190:         for (uint256 i = 0; i < poolCount; i++) {

190:         for (uint256 i = 0; i < poolCount; i++) {

201:         for (uint256 i = 0; i < poolCount; i++) {

201:         for (uint256 i = 0; i < poolCount; i++) {

257:         uint256 usersETH = TOTAL_STAKED_ETH - collateralETH;

261:         uint256 _userShareBeforeCommision = (_totalRewards * usersETH) / TOTAL_STAKED_ETH;

261:         uint256 _userShareBeforeCommision = (_totalRewards * usersETH) / TOTAL_STAKED_ETH;

263:         protocolShare = (protocolFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();

263:         protocolShare = (protocolFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();

265:         operatorShare = (_totalRewards * collateralETH) / TOTAL_STAKED_ETH;

265:         operatorShare = (_totalRewards * collateralETH) / TOTAL_STAKED_ETH;

266:         operatorShare += (operatorFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();

266:         operatorShare += (operatorFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();

266:         operatorShare += (operatorFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();

268:         userShare = _totalRewards - protocolShare - operatorShare;

268:         userShare = _totalRewards - protocolShare - operatorShare;

273:         for (uint256 i = 0; i < poolCount; i++) {

273:         for (uint256 i = 0; i < poolCount; i++) {

```

```solidity
File: SDCollateral.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import '../contracts/interfaces/IPoolUtils.sol';

6: import '../contracts/interfaces/IPoolUtils.sol';

6: import '../contracts/interfaces/IPoolUtils.sol';

7: import '../contracts/interfaces/SDCollateral/ISDCollateral.sol';

7: import '../contracts/interfaces/SDCollateral/ISDCollateral.sol';

7: import '../contracts/interfaces/SDCollateral/ISDCollateral.sol';

7: import '../contracts/interfaces/SDCollateral/ISDCollateral.sol';

8: import '../contracts/interfaces/SDCollateral/IAuction.sol';

8: import '../contracts/interfaces/SDCollateral/IAuction.sol';

8: import '../contracts/interfaces/SDCollateral/IAuction.sol';

8: import '../contracts/interfaces/SDCollateral/IAuction.sol';

9: import '../contracts/interfaces/IStaderOracle.sol';

9: import '../contracts/interfaces/IStaderOracle.sol';

9: import '../contracts/interfaces/IStaderOracle.sol';

11: import '@openzeppelin/contracts/utils/math/Math.sol';

11: import '@openzeppelin/contracts/utils/math/Math.sol';

11: import '@openzeppelin/contracts/utils/math/Math.sol';

11: import '@openzeppelin/contracts/utils/math/Math.sol';

12: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

14: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

14: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

14: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

14: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

45:         operatorSDBalance[operator] += _sdAmount;

62:         if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {

65:         operatorSDBalance[operator] -= _requestedSD;

96:         operatorSDBalance[_operator] -= sdSlashed;

148:         return convertETHToSD(poolThreshold.withdrawThreshold * validatorCount);

176:         _minSDToBond *= _numValidator;

190:         return (sdBalance >= minSDToBond ? 0 : minSDToBond - sdBalance);

199:         uint256 totalMinThreshold = validatorCount * convertETHToSD(poolThreshold.minThreshold);

200:         uint256 totalMaxThreshold = validatorCount * convertETHToSD(poolThreshold.maxThreshold);

207:         return (_sdAmount * sdPriceInETH) / staderConfig.getDecimals();

207:         return (_sdAmount * sdPriceInETH) / staderConfig.getDecimals();

212:         return (_ethAmount * staderConfig.getDecimals()) / sdPriceInETH;

212:         return (_ethAmount * staderConfig.getDecimals()) / sdPriceInETH;

```

```solidity
File: SocializingPool.sol

5: import './library/UtilLib.sol';

5: import './library/UtilLib.sol';

7: import './interfaces/ISocializingPool.sol';

7: import './interfaces/ISocializingPool.sol';

8: import './interfaces/IStaderStakePoolManager.sol';

8: import './interfaces/IStaderStakePoolManager.sol';

9: import './interfaces/IPermissionlessNodeRegistry.sol';

9: import './interfaces/IPermissionlessNodeRegistry.sol';

11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

12: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/utils/cryptography/MerkleProofUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/utils/cryptography/MerkleProofUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/utils/cryptography/MerkleProofUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/utils/cryptography/MerkleProofUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/utils/cryptography/MerkleProofUpgradeable.sol';

15: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

15: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

15: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

15: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

68:             _rewardsData.operatorETHRewards + _rewardsData.userETHRewards + _rewardsData.protocolETHRewards >

68:             _rewardsData.operatorETHRewards + _rewardsData.userETHRewards + _rewardsData.protocolETHRewards >

69:             address(this).balance - totalOperatorETHRewardsRemaining

75:             IERC20(staderConfig.getStaderToken()).balanceOf(address(this)) - totalOperatorSDRewardsRemaining

81:         totalOperatorETHRewardsRemaining += _rewardsData.operatorETHRewards;

82:         totalOperatorSDRewardsRemaining += _rewardsData.operatorSDRewards;

120:             totalOperatorETHRewardsRemaining -= totalAmountETH;

128:             totalOperatorSDRewardsRemaining -= totalAmountSD;

145:         for (uint256 i = 0; i < indexLength; i++) {

145:         for (uint256 i = 0; i < indexLength; i++) {

153:             _totalAmountSD += _amountSD[i];

154:             _totalAmountETH += _amountETH[i];

188:         index = lastReportedRewardsData.index + 1;

213:             return (initialBlock, initialBlock + cycleDuration);

217:         _startBlock = rewardsDataMap[_index - 1].reportingBlockNumber + 1;

217:         _startBlock = rewardsDataMap[_index - 1].reportingBlockNumber + 1;

222:             if (rewardsDataMap[_index - 1].reportingBlockNumber == 0) {

225:             _endBlock = rewardsDataMap[_index - 1].reportingBlockNumber + cycleDuration;

225:             _endBlock = rewardsDataMap[_index - 1].reportingBlockNumber + cycleDuration;

```

```solidity
File: StaderConfig.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './interfaces/IStaderConfig.sol';

6: import './interfaces/IStaderConfig.sol';

8: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

8: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

8: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

8: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

93:         setConstant(DECIMALS, 10**18);

93:         setConstant(DECIMALS, 10**18);

95:         setVariable(MIN_DEPOSIT_AMOUNT, 10**14);

95:         setVariable(MIN_DEPOSIT_AMOUNT, 10**14);

97:         setVariable(MIN_WITHDRAW_AMOUNT, 10**14);

97:         setVariable(MIN_WITHDRAW_AMOUNT, 10**14);

```

```solidity
File: StaderInsuranceFund.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './interfaces/IStaderConfig.sol';

6: import './interfaces/IStaderConfig.sol';

7: import './interfaces/IPermissionedPool.sol';

7: import './interfaces/IPermissionedPool.sol';

8: import './interfaces/IStaderInsuranceFund.sol';

8: import './interfaces/IStaderInsuranceFund.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

11: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

```

```solidity
File: StaderOracle.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './interfaces/IPoolUtils.sol';

6: import './interfaces/IPoolUtils.sol';

7: import './interfaces/IStaderOracle.sol';

7: import './interfaces/IStaderOracle.sol';

8: import './interfaces/ISocializingPool.sol';

8: import './interfaces/ISocializingPool.sol';

9: import './interfaces/INodeRegistry.sol';

9: import './interfaces/INodeRegistry.sol';

10: import './interfaces/IStaderStakePoolManager.sol';

10: import './interfaces/IStaderStakePoolManager.sol';

12: import '@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol';

12: import '@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol';

12: import '@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol';

12: import '@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol';

12: import '@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol';

13: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

15: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

15: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

15: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

15: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

26:     uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days

26:     uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days

26:     uint256 public constant MAX_ER_UPDATE_FREQUENCY = 7200 * 7; // 7 days

50:     bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency

50:     bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency

51:     bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key

51:     bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key

52:     bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key

52:     bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key

53:     bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key

53:     bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key

54:     bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key

54:     bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key

69:         erChangeLimit = 500; //5% deviation threshold

69:         erChangeLimit = 500; //5% deviation threshold

88:         trustedNodesCount++;

88:         trustedNodesCount++;

101:         trustedNodesCount--;

101:         trustedNodesCount--;

148:             submissionCount == trustedNodesCount / 2 + 1 &&

148:             submissionCount == trustedNodesCount / 2 + 1 &&

188:             erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY > block.number

255:         if ((submissionCount == trustedNodesCount / 2 + 1)) {

255:         if ((submissionCount == trustedNodesCount / 2 + 1)) {

290:         if ((submissionCount == (2 * trustedNodesCount) / 3 + 1)) {

290:         if ((submissionCount == (2 * trustedNodesCount) / 3 + 1)) {

290:         if ((submissionCount == (2 * trustedNodesCount) / 3 + 1)) {

304:         uint256 j = sdPrices.length - 1;

305:         while ((j >= 1) && (_sdPrice < sdPrices[j - 1])) {

306:             sdPrices[j] = sdPrices[j - 1];

307:             j--;

307:             j--;

314:         return (dataArray[(len - 1) / 2] + dataArray[len / 2]) / 2;

314:         return (dataArray[(len - 1) / 2] + dataArray[len / 2]) / 2;

314:         return (dataArray[(len - 1) / 2] + dataArray[len / 2]) / 2;

314:         return (dataArray[(len - 1) / 2] + dataArray[len / 2]) / 2;

314:         return (dataArray[(len - 1) / 2] + dataArray[len / 2]) / 2;

372:             submissionCount == trustedNodesCount / 2 + 1 &&

372:             submissionCount == trustedNodesCount / 2 + 1 &&

432:             submissionCount == trustedNodesCount / 2 + 1 &&

432:             submissionCount == trustedNodesCount / 2 + 1 &&

462:         if (_mapd.index != lastReportedMAPDIndex + 1) {

482:         if ((submissionCount == trustedNodesCount / 2 + 1)) {

482:         if ((submissionCount == trustedNodesCount / 2 + 1)) {

487:                 missedAttestationPenalty[pubkeyRoot]++;

487:                 missedAttestationPenalty[pubkeyRoot]++;

489:                     ++i;

489:                     ++i;

603:         return (block.number / updateFrequency) * updateFrequency;

603:         return (block.number / updateFrequency) * updateFrequency;

629:         submissionCountKeys[_submissionCountKey]++;

629:         submissionCountKeys[_submissionCountKey]++;

665:             !(newExchangeRate >= (currentExchangeRate * (ER_CHANGE_MAX_BPS - erChangeLimit)) / ER_CHANGE_MAX_BPS &&

665:             !(newExchangeRate >= (currentExchangeRate * (ER_CHANGE_MAX_BPS - erChangeLimit)) / ER_CHANGE_MAX_BPS &&

665:             !(newExchangeRate >= (currentExchangeRate * (ER_CHANGE_MAX_BPS - erChangeLimit)) / ER_CHANGE_MAX_BPS &&

666:                 newExchangeRate <= ((currentExchangeRate * (ER_CHANGE_MAX_BPS + erChangeLimit)) / ER_CHANGE_MAX_BPS))

666:                 newExchangeRate <= ((currentExchangeRate * (ER_CHANGE_MAX_BPS + erChangeLimit)) / ER_CHANGE_MAX_BPS))

666:                 newExchangeRate <= ((currentExchangeRate * (ER_CHANGE_MAX_BPS + erChangeLimit)) / ER_CHANGE_MAX_BPS))

```

```solidity
File: StaderStakePoolsManager.sol

5: import './library/UtilLib.sol';

5: import './library/UtilLib.sol';

7: import './ETHx.sol';

8: import './interfaces/IPoolUtils.sol';

8: import './interfaces/IPoolUtils.sol';

9: import './interfaces/IPoolSelector.sol';

9: import './interfaces/IPoolSelector.sol';

10: import './interfaces/IStaderConfig.sol';

10: import './interfaces/IStaderConfig.sol';

11: import './interfaces/IStaderOracle.sol';

11: import './interfaces/IStaderOracle.sol';

12: import './interfaces/IStaderPoolBase.sol';

12: import './interfaces/IStaderPoolBase.sol';

13: import './interfaces/IUserWithdrawalManager.sol';

13: import './interfaces/IUserWithdrawalManager.sol';

14: import './interfaces/IStaderStakePoolManager.sol';

14: import './interfaces/IStaderStakePoolManager.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

17: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

17: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

17: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

17: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

18: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

19: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

20: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

20: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

20: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

20: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

57:         excessETHDepositCoolDown = 3 * 7200;

192:         uint256 poolDepositSize = staderConfig.getStakedEthPerNode() - poolUtils.getCollateralETH(_poolId);

199:             (availableETHForNewDeposit / poolDepositSize)

207:         IStaderPoolBase(poolAddress).stakeUserETHToBeaconChain{value: selectedPoolCapacity * poolDepositSize}();

208:         emit ETHTransferredToPool(_poolId, poolAddress, selectedPoolCapacity * poolDepositSize);

216:         if (block.number < lastExcessETHDepositBlock + excessETHDepositCoolDown) {

232:         for (uint256 i = 0; i < poolCount; i++) {

232:         for (uint256 i = 0; i < poolCount; i++) {

238:             uint256 poolDepositSize = staderConfig.getStakedEthPerNode() -

243:             IStaderPoolBase(poolAddress).stakeUserETHToBeaconChain{value: validatorToDeposit * poolDepositSize}();

244:             emit ETHTransferredToPool(i, poolAddress, validatorToDeposit * poolDepositSize);

286:         Math.Rounding /*rounding*/

286:         Math.Rounding /*rounding*/

286:         Math.Rounding /*rounding*/

286:         Math.Rounding /*rounding*/

307:         Math.Rounding /*rounding*/

307:         Math.Rounding /*rounding*/

307:         Math.Rounding /*rounding*/

307:         Math.Rounding /*rounding*/

```

```solidity
File: UserWithdrawalManager.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

6: import './ETHx.sol';

7: import './interfaces/IStaderConfig.sol';

7: import './interfaces/IStaderConfig.sol';

8: import './interfaces/IStaderOracle.sol';

8: import './interfaces/IStaderOracle.sol';

9: import './interfaces/IStaderStakePoolManager.sol';

9: import './interfaces/IStaderStakePoolManager.sol';

10: import './interfaces/IUserWithdrawalManager.sol';

10: import './interfaces/IUserWithdrawalManager.sol';

12: import '@openzeppelin/contracts/utils/math/Math.sol';

12: import '@openzeppelin/contracts/utils/math/Math.sol';

12: import '@openzeppelin/contracts/utils/math/Math.sol';

12: import '@openzeppelin/contracts/utils/math/Math.sol';

13: import '@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol';

13: import '@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

14: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

15: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

15: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

15: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

15: import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';

16: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

16: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

16: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

16: import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

42:         address payable owner; // address that can claim eth on behalf of this request

42:         address payable owner; // address that can claim eth on behalf of this request

43:         uint256 ethXAmount; //amount of ethX share locked for withdrawal

43:         uint256 ethXAmount; //amount of ethX share locked for withdrawal

44:         uint256 ethExpected; //eth requested according to given share and exchangeRate

44:         uint256 ethExpected; //eth requested according to given share and exchangeRate

45:         uint256 ethFinalized; // final eth for claiming according to finalize exchange rate

45:         uint256 ethFinalized; // final eth for claiming according to finalize exchange rate

46:         uint256 requestBlock; // block number of withdraw request

46:         uint256 requestBlock; // block number of withdraw request

101:         if (requestIdsByUserAddress[_owner].length + 1 > maxNonRedeemedUserRequestCount) {

105:         ethRequestedForWithdraw += assets;

109:         nextRequestId++;

109:         nextRequestId++;

110:         return nextRequestId - 1;

127:         uint256 maxRequestIdToFinalize = Math.min(nextRequestId, nextRequestIdToFinalize + finalizationBatchLimit) - 1;

127:         uint256 maxRequestIdToFinalize = Math.min(nextRequestId, nextRequestIdToFinalize + finalizationBatchLimit) - 1;

136:             uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);

136:             uint256 minEThRequiredToFinalizeRequest = Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS);

138:                 (ethToSendToFinalizeRequest + minEThRequiredToFinalizeRequest > pooledETH) ||

139:                 (userWithdrawInfo.requestBlock + staderConfig.getMinBlockDelayToFinalizeWithdrawRequest() >

145:             ethRequestedForWithdraw -= requiredEth;

146:             lockedEthXToBurn += lockedEthX;

147:             ethToSendToFinalizeRequest += minEThRequiredToFinalizeRequest;

149:                 ++requestId;

149:                 ++requestId;

212:                 requestIds[i] = requestIds[userRequestCount - 1];

217:                 ++i;

217:                 ++i;

```

```solidity
File: ValidatorWithdrawalVault.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

5: import './library/ValidatorStatus.sol';

5: import './library/ValidatorStatus.sol';

7: import './VaultProxy.sol';

8: import './interfaces/IPenalty.sol';

8: import './interfaces/IPenalty.sol';

9: import './interfaces/IPoolUtils.sol';

9: import './interfaces/IPoolUtils.sol';

10: import './interfaces/INodeRegistry.sol';

10: import './interfaces/INodeRegistry.sol';

11: import './interfaces/IStaderStakePoolManager.sol';

11: import './interfaces/IStaderStakePoolManager.sol';

12: import './interfaces/IValidatorWithdrawalVault.sol';

12: import './interfaces/IValidatorWithdrawalVault.sol';

13: import './interfaces/SDCollateral/ISDCollateral.sol';

13: import './interfaces/SDCollateral/ISDCollateral.sol';

13: import './interfaces/SDCollateral/ISDCollateral.sol';

14: import './interfaces/IOperatorRewardsCollector.sol';

14: import './interfaces/IOperatorRewardsCollector.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

16: import '@openzeppelin/contracts/utils/math/Math.sol';

71:         uint256 userShare = userSharePrelim + penaltyAmount;

72:         operatorShare = operatorShare - penaltyAmount;

97:         uint256 collateralETH = getCollateralETH(poolId, staderConfig); // 0, incase of permissioned NOs

97:         uint256 collateralETH = getCollateralETH(poolId, staderConfig); // 0, incase of permissioned NOs

98:         uint256 usersETH = TOTAL_STAKED_ETH - collateralETH;

108:             _operatorShare = contractBalance - _userShare;

111:             totalRewards = contractBalance - TOTAL_STAKED_ETH;

119:             _userShare += userReward;

120:             _operatorShare += operatorReward;

121:             _protocolShare += protocolReward;

```

```solidity
File: VaultProxy.sol

4: import './library/UtilLib.sol';

4: import './library/UtilLib.sol';

5: import './interfaces/IVaultProxy.sol';

5: import './interfaces/IVaultProxy.sol';

13:     uint256 public override id; //validatorId or operatorId based on vault type

13:     uint256 public override id; //validatorId or operatorId based on vault type

```

```solidity
File: factory/VaultFactory.sol

4: import '../library/UtilLib.sol';

4: import '../library/UtilLib.sol';

5: import '../VaultProxy.sol';

6: import '../interfaces/IVaultFactory.sol';

6: import '../interfaces/IVaultFactory.sol';

7: import '../interfaces/IStaderConfig.sol';

7: import '../interfaces/IStaderConfig.sol';

9: import '@openzeppelin/contracts-upgradeable/proxy/ClonesUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/proxy/ClonesUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/proxy/ClonesUpgradeable.sol';

9: import '@openzeppelin/contracts-upgradeable/proxy/ClonesUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

10: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

```

```solidity
File: interfaces/INodeELRewardVault.sol

4: import './IStaderConfig.sol';

```

```solidity
File: interfaces/INodeRegistry.sol

4: import '../library/ValidatorStatus.sol';

4: import '../library/ValidatorStatus.sol';

7:     ValidatorStatus status; // status of validator

7:     ValidatorStatus status; // status of validator

8:     bytes pubkey; //pubkey of the validator

8:     bytes pubkey; //pubkey of the validator

9:     bytes preDepositSignature; //signature for 1 ETH deposit on beacon chain

9:     bytes preDepositSignature; //signature for 1 ETH deposit on beacon chain

10:     bytes depositSignature; //signature for 31 ETH deposit on beacon chain

10:     bytes depositSignature; //signature for 31 ETH deposit on beacon chain

11:     address withdrawVaultAddress; //withdrawal vault address of validator

11:     address withdrawVaultAddress; //withdrawal vault address of validator

12:     uint256 operatorId; // stader network assigned Id

12:     uint256 operatorId; // stader network assigned Id

13:     uint256 depositBlock; // block number of the 31ETH deposit

13:     uint256 depositBlock; // block number of the 31ETH deposit

14:     uint256 withdrawnBlock; //block number when oracle report validator as withdrawn

14:     uint256 withdrawnBlock; //block number when oracle report validator as withdrawn

18:     bool active; // operator status

18:     bool active; // operator status

19:     bool optedForSocializingPool; // operator opted for socializing pool

19:     bool optedForSocializingPool; // operator opted for socializing pool

20:     string operatorName; // name of the operator

20:     string operatorName; // name of the operator

21:     address payable operatorRewardAddress; //Eth1 address of node for reward

21:     address payable operatorRewardAddress; //Eth1 address of node for reward

22:     address operatorAddress; //address of operator to interact with stader

22:     address operatorAddress; //address of operator to interact with stader

```

```solidity
File: interfaces/IPermissionedNodeRegistry.sol

4: import '../library/ValidatorStatus.sol';

4: import '../library/ValidatorStatus.sol';

5: import './INodeRegistry.sol';

```

```solidity
File: interfaces/IPermissionlessNodeRegistry.sol

4: import '../library/ValidatorStatus.sol';

4: import '../library/ValidatorStatus.sol';

5: import './INodeRegistry.sol';

```

```solidity
File: interfaces/IPoolUtils.sol

4: import './INodeRegistry.sol';

50:     function getProtocolFee(uint8 _poolId) external view returns (uint256); // returns the protocol fee (0-10000)

50:     function getProtocolFee(uint8 _poolId) external view returns (uint256); // returns the protocol fee (0-10000)

50:     function getProtocolFee(uint8 _poolId) external view returns (uint256); // returns the protocol fee (0-10000)

52:     function getOperatorFee(uint8 _poolId) external view returns (uint256); // returns the operator fee (0-10000)

52:     function getOperatorFee(uint8 _poolId) external view returns (uint256); // returns the operator fee (0-10000)

52:     function getOperatorFee(uint8 _poolId) external view returns (uint256); // returns the operator fee (0-10000)

54:     function getTotalActiveValidatorCount() external view returns (uint256); //returns total active validators across all pools

54:     function getTotalActiveValidatorCount() external view returns (uint256); //returns total active validators across all pools

56:     function getActiveValidatorCountByPool(uint8 _poolId) external view returns (uint256); // returns the total number of active validators in a specific pool

56:     function getActiveValidatorCountByPool(uint8 _poolId) external view returns (uint256); // returns the total number of active validators in a specific pool

58:     function getQueuedValidatorCountByPool(uint8 _poolId) external view returns (uint256); // returns the total number of queued validators in a specific pool

58:     function getQueuedValidatorCountByPool(uint8 _poolId) external view returns (uint256); // returns the total number of queued validators in a specific pool

```

```solidity
File: interfaces/ISocializingPool.sol

5: import './IStaderConfig.sol';

```

```solidity
File: interfaces/IStaderOracle.sol

4: import '../library/ValidatorStatus.sol';

4: import '../library/ValidatorStatus.sol';

6: import './ISocializingPool.sol';

7: import './IStaderConfig.sol';

```

```solidity
File: interfaces/IStaderPoolBase.sol

4: import './INodeRegistry.sol';

23:     function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external; // sets the commission fees, protocol and operator

23:     function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external; // sets the commission fees, protocol and operator

27:     function protocolFee() external view returns (uint256); // returns the protocol fee

27:     function protocolFee() external view returns (uint256); // returns the protocol fee

29:     function operatorFee() external view returns (uint256); // returns the operator fee

29:     function operatorFee() external view returns (uint256); // returns the operator fee

31:     function getTotalActiveValidatorCount() external view returns (uint256); // returns the total number of active validators across all operators

31:     function getTotalActiveValidatorCount() external view returns (uint256); // returns the total number of active validators across all operators

33:     function getTotalQueuedValidatorCount() external view returns (uint256); // returns the total number of queued validators across all operators

33:     function getTotalQueuedValidatorCount() external view returns (uint256); // returns the total number of queued validators across all operators

```

```solidity
File: interfaces/IValidatorWithdrawalVault.sol

4: import './IStaderConfig.sol';

```

```solidity
File: interfaces/IVaultProxy.sol

4: import './IStaderConfig.sol';

```

```solidity
File: interfaces/SDCollateral/IAuction.sol

4: import '../IStaderConfig.sol';

```

```solidity
File: interfaces/SDCollateral/ISDCollateral.sol

4: import '../IStaderConfig.sol';

```

```solidity
File: library/UtilLib.sol

4: import '../interfaces/IStaderConfig.sol';

4: import '../interfaces/IStaderConfig.sol';

5: import '../interfaces/INodeRegistry.sol';

5: import '../interfaces/INodeRegistry.sol';

6: import '../interfaces/IPoolUtils.sol';

6: import '../interfaces/IPoolUtils.sol';

7: import '../interfaces/IVaultProxy.sol';

7: import '../interfaces/IVaultProxy.sol';

163:             : (totalETHBalance * DECIMALS) / totalETHXSupply;

163:             : (totalETHBalance * DECIMALS) / totalETHXSupply;

```

### <a name="GAS-7"></a>[GAS-7] Don't initialize variables with default value

*Instances (13)*:
```solidity
File: PermissionedNodeRegistry.sol

91:         for (uint256 i = 0; i < permissionedNosLength; i++) {

597:         uint256 validatorCount = 0;

```

```solidity
File: PermissionlessNodeRegistry.sol

503:         uint256 validatorCount = 0;

572:         uint256 optOutOperatorCount = 0;

```

```solidity
File: PoolSelector.sol

130:         for (uint256 i = 0; i < poolTargetLength; i++) {

```

```solidity
File: PoolUtils.sol

104:         for (uint256 i = 0; i < poolCount; i++) {

168:         for (uint256 i = 0; i < poolCount; i++) {

179:         for (uint256 i = 0; i < poolCount; i++) {

190:         for (uint256 i = 0; i < poolCount; i++) {

201:         for (uint256 i = 0; i < poolCount; i++) {

273:         for (uint256 i = 0; i < poolCount; i++) {

```

```solidity
File: SocializingPool.sol

145:         for (uint256 i = 0; i < indexLength; i++) {

```

```solidity
File: StaderStakePoolsManager.sol

232:         for (uint256 i = 0; i < poolCount; i++) {

```

### <a name="GAS-8"></a>[GAS-8] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (78)*:
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

656:     function onlyPreDepositValidator(bytes calldata _pubkey) external view override {

720:     function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {

752:     function onlyPreDepositValidator(uint256 _validatorId) internal view {

```

```solidity
File: PermissionedPool.sol

248:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: PermissionlessNodeRegistry.sol

354:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

680:     function onlyActiveOperator(address _operAddr) internal view returns (uint256 _operatorId) {

712:     function onlyInitializedValidator(uint256 _validatorId) internal view {

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

84:     function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {

91:     function getProtocolFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {

96:     function getOperatorFee(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {

157:     function getCollateralETH(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (uint256) {

162:     function getNodeRegistry(uint8 _poolId) public view override onlyExistingPoolId(_poolId) returns (address) {

215:     function onlyValidName(string calldata _name) external view {

225:     function onlyValidKeys(

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

118:     function updateRewardsThreshold(uint256 _rewardsThreshold) external onlyRole(MANAGER) {

126:     function updateMinDepositAmount(uint256 _minDepositAmount) external onlyRole(MANAGER) {

135:     function updateMaxDepositAmount(uint256 _maxDepositAmount) external onlyRole(MANAGER) {

144:     function updateMinWithdrawAmount(uint256 _minWithdrawAmount) external onlyRole(DEFAULT_ADMIN_ROLE) {

153:     function updateMaxWithdrawAmount(uint256 _maxWithdrawAmount) external onlyRole(DEFAULT_ADMIN_ROLE) {

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

268:     function updateNodeELRewardImplementation(address _nodeELRewardVaultImpl) external onlyRole(DEFAULT_ADMIN_ROLE) {

279:     function updateETHBalancePORFeedProxy(address _ethBalanceProxy) external onlyRole(DEFAULT_ADMIN_ROLE) {

283:     function updateETHXSupplyPORFeedProxy(address _ethXSupplyProxy) external onlyRole(DEFAULT_ADMIN_ROLE) {

287:     function updateStaderToken(address _staderToken) external onlyRole(DEFAULT_ADMIN_ROLE) {

291:     function updateETHxToken(address _ethX) external onlyRole(DEFAULT_ADMIN_ROLE) {

500:     function onlyStaderContract(address _addr, bytes32 _contractName) external view returns (bool) {

504:     function onlyManagerRole(address account) public view override returns (bool) {

508:     function onlyOperatorRole(address account) public view override returns (bool) {

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

86:     function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {

93:     function updateVaultProxyAddress(address _vaultProxyImpl) external override onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: interfaces/IPermissionedNodeRegistry.sol

35:     function onlyPreDepositValidator(bytes calldata _pubkey) external view;

```

```solidity
File: interfaces/IPoolUtils.sol

76:     function onlyValidName(string calldata _name) external;

78:     function onlyValidKeys(

```

```solidity
File: interfaces/IStaderConfig.sol

158:     function onlyStaderContract(address _addr, bytes32 _contractName) external view returns (bool);

160:     function onlyManagerRole(address account) external view returns (bool);

162:     function onlyOperatorRole(address account) external view returns (bool);

```

```solidity
File: library/UtilLib.sol

26:     function onlyManagerRole(address _addr, IStaderConfig _staderConfig) internal view {

32:     function onlyOperatorRole(address _addr, IStaderConfig _staderConfig) internal view {

39:     function onlyStaderContract(

82:     function onlyValidatorWithdrawVault(

```

### <a name="GAS-9"></a>[GAS-9] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (33)*:
```solidity
File: Auction.sol

59:         nextLot++;

```

```solidity
File: PermissionedNodeRegistry.sol

91:         for (uint256 i = 0; i < permissionedNosLength; i++) {

176:             nextValidatorId++;

537:                 totalNonWithdrawnKeyCount++;

598:         for (uint256 i = startIndex; i < endIndex; i++) {

601:                 validatorCount++;

637:         for (uint256 i = startIndex; i < endIndex; i++) {

665:         nextOperatorId++;

666:         totalActiveOperatorCount++;

770:         totalActiveOperatorCount++;

```

```solidity
File: PermissionedPool.sol

112:                 index++

```

```solidity
File: PermissionlessNodeRegistry.sol

162:             nextValidatorId++;

418:                 totalNonWithdrawnKeyCount++;

504:         for (uint256 i = startIndex; i < endIndex; i++) {

507:                 validatorCount++;

543:         for (uint256 i = startIndex; i < endIndex; i++) {

573:         for (uint256 i = startIndex; i < endIndex; i++) {

576:                 optOutOperatorCount++;

612:         nextOperatorId++;

621:         validatorQueueSize++;

```

```solidity
File: PoolSelector.sol

130:         for (uint256 i = 0; i < poolTargetLength; i++) {

```

```solidity
File: PoolUtils.sol

104:         for (uint256 i = 0; i < poolCount; i++) {

168:         for (uint256 i = 0; i < poolCount; i++) {

179:         for (uint256 i = 0; i < poolCount; i++) {

190:         for (uint256 i = 0; i < poolCount; i++) {

201:         for (uint256 i = 0; i < poolCount; i++) {

273:         for (uint256 i = 0; i < poolCount; i++) {

```

```solidity
File: SocializingPool.sol

145:         for (uint256 i = 0; i < indexLength; i++) {

```

```solidity
File: StaderOracle.sol

88:         trustedNodesCount++;

487:                 missedAttestationPenalty[pubkeyRoot]++;

629:         submissionCountKeys[_submissionCountKey]++;

```

```solidity
File: StaderStakePoolsManager.sol

232:         for (uint256 i = 0; i < poolCount; i++) {

```

```solidity
File: UserWithdrawalManager.sol

109:         nextRequestId++;

```

### <a name="GAS-10"></a>[GAS-10] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (63)*:
```solidity
File: Auction.sol

22:     uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours

```

```solidity
File: ETHx.sol

21:     bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');

22:     bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');

```

```solidity
File: PermissionedNodeRegistry.sol

31:     uint8 public constant override POOL_ID = 2;

```

```solidity
File: PermissionedPool.sol

33:     uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

```

```solidity
File: PermissionlessNodeRegistry.sol

30:     uint8 public constant override POOL_ID = 1;

42:     uint256 public constant override FRONT_RUN_PENALTY = 3 ether;

43:     uint256 public constant COLLATERAL_ETH = 4 ether;

```

```solidity
File: PermissionlessPool.sol

23:     uint256 public constant DEPOSIT_NODE_BOND = 3 ether;

31:     uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

```

```solidity
File: PoolSelector.sol

21:     uint256 public constant POOL_WEIGHTS_SUM = 10000;

```

```solidity
File: StaderConfig.sol

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
File: StaderOracle.sol

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
File: factory/VaultFactory.sol

16:     bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');

```

### <a name="GAS-11"></a>[GAS-11] Use shift Right/Left instead of division/multiplication if possible

*Instances (6)*:
```solidity
File: StaderOracle.sol

148:             submissionCount == trustedNodesCount / 2 + 1 &&

255:         if ((submissionCount == trustedNodesCount / 2 + 1)) {

314:         return (dataArray[(len - 1) / 2] + dataArray[len / 2]) / 2;

372:             submissionCount == trustedNodesCount / 2 + 1 &&

432:             submissionCount == trustedNodesCount / 2 + 1 &&

482:         if ((submissionCount == trustedNodesCount / 2 + 1)) {

```

### <a name="GAS-12"></a>[GAS-12] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (13)*:
```solidity
File: Auction.sol

109:         if (lotItem.highestBidAmount > 0) revert LotWasAuctioned();

```

```solidity
File: PermissionedNodeRegistry.sol

204:         bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);

299:         if (totalDefectedKeys > 0) {

```

```solidity
File: PermissionlessNodeRegistry.sol

209:         if (frontRunValidatorsLength > 0) {

305:             if (address(feeRecipientAddress).balance > 0) {

```

```solidity
File: SocializingPool.sol

119:         if (totalAmountETH > 0) {

127:         if (totalAmountSD > 0) {

```

```solidity
File: StaderOracle.sol

121:         if (_exchangeRate.reportingBlockNumber % updateFrequencyMap[ETHX_ER_UF] > 0) {

274:         if (_sdPriceData.reportingBlockNumber % updateFrequencyMap[SD_PRICE_UF] > 0) {

328:         if (_validatorStats.reportingBlockNumber % updateFrequencyMap[VALIDATOR_STATS_UF] > 0) {

403:         if (_withdrawnValidators.reportingBlockNumber % updateFrequencyMap[WITHDRAWN_VALIDATORS_UF] > 0) {

```

```solidity
File: StaderStakePoolsManager.sol

330:             (totalAssets() > 0 ||

```

```solidity
File: ValidatorWithdrawalVault.sol

115:         if (totalRewards > 0) {

```

### <a name="GAS-13"></a>[GAS-13] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (14)*:
```solidity
File: library/UtilLib.sol

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


