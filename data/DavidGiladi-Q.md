## Reentrancy vulnerabilities
- Severity: Low
- Confidence: Medium

### Description

Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy).
Only report reentrancies leading to out-of-order events.

<details>

<summary>
There are 21 instances of this issue:

</summary>

###
- Reentrancy in `File: contracts/PermissionedNodeRegistry.sol#311-331` <br> [PermissionedNodeRegistry.withdrawnValidators(bytes[])](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L311-L331):
	External calls:
	- `File: contracts/PermissionedNodeRegistry.sol#324` <br> [IValidatorWithdrawalVault(validatorRegistry[validatorId].withdrawVaultAddress).settleFunds()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L324)
	Event emitted after the call(s):
	- `File: contracts/PermissionedNodeRegistry.sol#749` <br> [DecreasedTotalActiveValidatorCount(totalActiveValidatorCount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L749)
		- `File: contracts/PermissionedNodeRegistry.sol#330` <br> [decreaseTotalActiveValidatorCount(withdrawnValidatorCount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L330)
	- `File: contracts/PermissionedNodeRegistry.sol#325` <br> [ValidatorWithdrawn(_pubkeys[i], validatorId)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L325)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L311-L331


- Reentrancy in `File: contracts/PermissionlessNodeRegistry.sol#366-374` <br> [PermissionlessNodeRegistry.updateOperatorDetails(string,address)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L366-L374):
	External calls:
	- `File: contracts/PermissionlessNodeRegistry.sol#367` <br> [IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L367)
	Event emitted after the call(s):
	- `File: contracts/PermissionlessNodeRegistry.sol#373` <br> [UpdatedOperatorDetails(msg.sender, _operatorName, _rewardAddress)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L373)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L366-L374


- Reentrancy in `File: contracts/UserWithdrawalManager.sol#95-111` <br> [UserWithdrawalManager.requestWithdraw(uint256,address)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L95-L111):
	External calls:
	- `File: contracts/UserWithdrawalManager.sol#104` <br> [IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L104)
	Event emitted after the call(s):
	- `File: contracts/UserWithdrawalManager.sol#108` <br> [WithdrawRequestReceived(msg.sender, _owner, nextRequestId, _ethXAmount, assets)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L108)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L95-L111


- Reentrancy in `File: contracts/PermissionedNodeRegistry.sol#401-409` <br> [PermissionedNodeRegistry.updateOperatorDetails(string,address)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L401-L409):
	External calls:
	- `File: contracts/PermissionedNodeRegistry.sol#402` <br> [IPoolUtils(staderConfig.getPoolUtils()).onlyValidName(_operatorName)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L402)
	Event emitted after the call(s):
	- `File: contracts/PermissionedNodeRegistry.sol#408` <br> [UpdatedOperatorDetails(msg.sender, _operatorName, _rewardAddress)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L408)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L401-L409


- Reentrancy in `File: contracts/PermissionlessNodeRegistry.sol#89-116` <br> [PermissionlessNodeRegistry.onboardNodeOperator(bool,string,address)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L89-L116):
	External calls:
	- `File: contracts/PermissionlessNodeRegistry.sol#98` <br> [IPoolUtils(poolUtils).onlyValidName(_operatorName)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L98)
	- `File: contracts/PermissionlessNodeRegistry.sol#106-109` <br> [IVaultFactory(staderConfig.getVaultFactory()).deployNodeELRewardVault(

            POOL_ID,

            nextOperatorId

        )](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L106-L109)
	Event emitted after the call(s):
	- `File: contracts/PermissionlessNodeRegistry.sol#614` <br> [OnboardedOperator(msg.sender, _operatorRewardAddress, nextOperatorId - 1, _optInForSocializingPool)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L614)
		- `File: contracts/PermissionlessNodeRegistry.sol#114` <br> [onboardOperator(_optInForSocializingPool, _operatorName, _operatorRewardAddress)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L114)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L89-L116


- Reentrancy in `File: contracts/ValidatorWithdrawalVault.sol#30-52` <br> [ValidatorWithdrawalVault.distributeRewards()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L30-L52):
	External calls:
	- `File: contracts/ValidatorWithdrawalVault.sol#46` <br> [IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L46)
	- `File: contracts/ValidatorWithdrawalVault.sol#47` <br> [UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L47)
	- `File: contracts/ValidatorWithdrawalVault.sol#48-50` <br> [IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(

            getOperatorAddress(poolId, validatorId, staderConfig)

        )](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L48-L50)
	External calls sending eth:
	- `File: contracts/ValidatorWithdrawalVault.sol#46` <br> [IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L46)
	- `File: contracts/ValidatorWithdrawalVault.sol#48-50` <br> [IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(

            getOperatorAddress(poolId, validatorId, staderConfig)

        )](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L48-L50)
	Event emitted after the call(s):
	- `File: contracts/ValidatorWithdrawalVault.sol#51` <br> [DistributedRewards(userShare, operatorShare, protocolShare)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L51)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L30-L52


- Reentrancy in `File: contracts/PermissionlessNodeRegistry.sol#287-313` <br> [PermissionlessNodeRegistry.changeSocializingPoolState(bool)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L287-L313):
	External calls:
	- `File: contracts/PermissionlessNodeRegistry.sol#306` <br> [INodeELRewardVault(feeRecipientAddress).withdraw()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L306)
	Event emitted after the call(s):
	- `File: contracts/PermissionlessNodeRegistry.sol#312` <br> [UpdatedSocializingPoolState(operatorId, _optInForSocializingPool, block.number)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L312)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L287-L313


- Reentrancy in `File: contracts/StaderStakePoolsManager.sol#315-323` <br> [StaderStakePoolsManager._deposit(address,address,uint256,uint256)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L315-L323):
	External calls:
	- `File: contracts/StaderStakePoolsManager.sol#321` <br> [ETHx(staderConfig.getETHxToken()).mint(_receiver, _shares)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L321)
	Event emitted after the call(s):
	- `File: contracts/StaderStakePoolsManager.sol#322` <br> [Deposited(_caller, _receiver, _assets, _shares)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L322)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L315-L323


- Reentrancy in `File: contracts/ValidatorWithdrawalVault.sol#54-83` <br> [ValidatorWithdrawalVault.settleFunds()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L54-L83):
	External calls:
	- `File: contracts/ValidatorWithdrawalVault.sol#64` <br> [getUpdatedPenaltyAmount(poolId, validatorId, staderConfig)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L64)
		- `File: contracts/ValidatorWithdrawalVault.sol#148` <br> [IPenalty(_staderConfig.getPenaltyContract()).updateTotalPenaltyAmount(pubkeyArray)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L148)
	- `File: contracts/ValidatorWithdrawalVault.sol#67` <br> [ISDCollateral(staderConfig.getSDCollateral()).slashValidatorSD(validatorId, poolId)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L67)
	- `File: contracts/ValidatorWithdrawalVault.sol#76` <br> [IPenalty(staderConfig.getPenaltyContract()).markValidatorSettled(poolId, validatorId)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L76)
	- `File: contracts/ValidatorWithdrawalVault.sol#77` <br> [IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L77)
	- `File: contracts/ValidatorWithdrawalVault.sol#78` <br> [UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L78)
	- `File: contracts/ValidatorWithdrawalVault.sol#79-81` <br> [IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(

            getOperatorAddress(poolId, validatorId, staderConfig)

        )](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L79-L81)
	External calls sending eth:
	- `File: contracts/ValidatorWithdrawalVault.sol#77` <br> [IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L77)
	- `File: contracts/ValidatorWithdrawalVault.sol#79-81` <br> [IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(

            getOperatorAddress(poolId, validatorId, staderConfig)

        )](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L79-L81)
	Event emitted after the call(s):
	- `File: contracts/ValidatorWithdrawalVault.sol#82` <br> [SettledFunds(userShare, operatorShare, protocolShare)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L82)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L54-L83


- Reentrancy in `File: contracts/PermissionedNodeRegistry.sol#140-181` <br> [PermissionedNodeRegistry.addValidatorKeys(bytes[],bytes[],bytes[])](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L140-L181):
	External calls:
	- `File: contracts/PermissionedNodeRegistry.sol#156` <br> [IPoolUtils(poolUtils).onlyValidKeys(_pubkey[i], _preDepositSignature[i], _depositSignature[i])](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L156)
	- `File: contracts/PermissionedNodeRegistry.sol#157-162` <br> [IVaultFactory(vaultFactory).deployWithdrawVault(

                POOL_ID,

                operatorId,

                operatorTotalKeys + i, //operator totalKeys

                nextValidatorId

            )](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L157-L162)
	Event emitted after the call(s):
	- `File: contracts/PermissionedNodeRegistry.sol#175` <br> [AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L175)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L140-L181


- Reentrancy in `File: contracts/factory/VaultFactory.sol#48-60` <br> [VaultFactory.deployNodeELRewardVault(uint8,uint256)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L48-L60):
	External calls:
	- `File: contracts/factory/VaultFactory.sol#56` <br> [VaultProxy(payable(nodeELRewardVaultAddress)).initialise(false, _poolId, _operatorId, address(staderConfig))](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L56)
	Event emitted after the call(s):
	- `File: contracts/factory/VaultFactory.sol#58` <br> [NodeELRewardVaultCreated(nodeELRewardVaultAddress)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L58)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L48-L60


- Reentrancy in `File: contracts/PermissionlessNodeRegistry.sol#241-261` <br> [PermissionlessNodeRegistry.withdrawnValidators(bytes[])](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L241-L261):
	External calls:
	- `File: contracts/PermissionlessNodeRegistry.sol#254` <br> [IValidatorWithdrawalVault(validatorRegistry[validatorId].withdrawVaultAddress).settleFunds()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L254)
	Event emitted after the call(s):
	- `File: contracts/PermissionlessNodeRegistry.sol#709` <br> [DecreasedTotalActiveValidatorCount(totalActiveValidatorCount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L709)
		- `File: contracts/PermissionlessNodeRegistry.sol#260` <br> [decreaseTotalActiveValidatorCount(withdrawnValidatorCount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L260)
	- `File: contracts/PermissionlessNodeRegistry.sol#255` <br> [ValidatorWithdrawn(_pubkeys[i], validatorId)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L255)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L241-L261


- Reentrancy in `File: contracts/SDCollateral.sol#58-73` <br> [SDCollateral.withdraw(uint256)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L58-L73):
	External calls:
	- `File: contracts/SDCollateral.sol#68` <br> [!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L68)
	Event emitted after the call(s):
	- `File: contracts/SDCollateral.sol#72` <br> [SDWithdrawn(operator, _requestedSD)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L72)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L58-L73


- Reentrancy in `File: contracts/PermissionedNodeRegistry.sol#106-131` <br> [PermissionedNodeRegistry.onboardNodeOperator(string,address)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L106-L131):
	External calls:
	- `File: contracts/PermissionedNodeRegistry.sol#116` <br> [IPoolUtils(poolUtils).onlyValidName(_operatorName)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L116)
	Event emitted after the call(s):
	- `File: contracts/PermissionedNodeRegistry.sol#667` <br> [OnboardedOperator(msg.sender, _operatorRewardAddress, nextOperatorId - 1)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L667)
		- `File: contracts/PermissionedNodeRegistry.sol#129` <br> [onboardOperator(_operatorName, _operatorRewardAddress)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L129)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L106-L131


- Reentrancy in `File: contracts/factory/VaultFactory.sol#34-46` <br> [VaultFactory.deployWithdrawVault(uint8,uint256,uint256,uint256)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L34-L46):
	External calls:
	- `File: contracts/factory/VaultFactory.sol#42` <br> [VaultProxy(payable(withdrawVaultAddress)).initialise(true, _poolId, _validatorId, address(staderConfig))](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L42)
	Event emitted after the call(s):
	- `File: contracts/factory/VaultFactory.sol#44` <br> [WithdrawVaultCreated(withdrawVaultAddress)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L44)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L34-L46


- Reentrancy in `File: contracts/Auction.sol#80-91` <br> [Auction.claimSD(uint256)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L80-L91):
	External calls:
	- `File: contracts/Auction.sol#87` <br> [!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L87)
	Event emitted after the call(s):
	- `File: contracts/Auction.sol#90` <br> [SDClaimed(lotId, lotItem.highestBidder, lotItem.sdAmount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L90)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L80-L91


- Reentrancy in `File: contracts/NodeELRewardVault.sol#24-45` <br> [NodeELRewardVault.withdraw()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L24-L45):
	External calls:
	- `File: contracts/NodeELRewardVault.sol#36` <br> [IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{value: userShare}()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L36)
	- `File: contracts/NodeELRewardVault.sol#38` <br> [UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L38)
	- `File: contracts/NodeELRewardVault.sol#40-42` <br> [IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(

            operator

        )](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L40-L42)
	External calls sending eth:
	- `File: contracts/NodeELRewardVault.sol#36` <br> [IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveExecutionLayerRewards{value: userShare}()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L36)
	- `File: contracts/NodeELRewardVault.sol#40-42` <br> [IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(

            operator

        )](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L40-L42)
	Event emitted after the call(s):
	- `File: contracts/NodeELRewardVault.sol#44` <br> [Withdrawal(protocolShare, operatorShare, userShare)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L44)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L24-L45


- Reentrancy in `File: contracts/SDCollateral.sol#43-52` <br> [SDCollateral.depositSDAsCollateral(uint256)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L43-L52):
	External calls:
	- `File: contracts/SDCollateral.sol#47` <br> [!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L47)
	Event emitted after the call(s):
	- `File: contracts/SDCollateral.sol#51` <br> [SDDeposited(operator, _sdAmount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L51)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L43-L52


- Reentrancy in `File: contracts/Auction.sol#106-118` <br> [Auction.extractNonBidSD(uint256)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L106-L118):
	External calls:
	- `File: contracts/Auction.sol#114` <br> [!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L114)
	Event emitted after the call(s):
	- `File: contracts/Auction.sol#117` <br> [UnsuccessfulSDAuctionExtracted(lotId, _sdAmount, staderConfig.getStaderTreasury())](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L117)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L106-L118


- Reentrancy in `File: contracts/Auction.sol#48-60` <br> [Auction.createLot(uint256)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L48-L60):
	External calls:
	- `File: contracts/Auction.sol#55` <br> [!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L55)
	Event emitted after the call(s):
	- `File: contracts/Auction.sol#58` <br> [LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L58)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L48-L60


- Reentrancy in `File: contracts/OperatorRewardsCollector.sol#46-54` <br> [OperatorRewardsCollector.claim()](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L46-L54):
	External calls:
	- `File: contracts/OperatorRewardsCollector.sol#52` <br> [UtilLib.sendValue(operatorRewardsAddr, amount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L52)
	Event emitted after the call(s):
	- `File: contracts/OperatorRewardsCollector.sol#53` <br> [Claimed(operatorRewardsAddr, amount)](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L53)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L46-L54
