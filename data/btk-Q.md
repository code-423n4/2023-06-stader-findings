# Protocol Overview

The purpose of the ETHx protocol is to create a multi-pool architecture for node operations that emphasizes decentralization, scalability, and resilience. It aims to democratize node operations and adapt to increasing demand. The protocol includes both a permissionless pool, open to anyone who wants to participate and operate nodes, and a permissioned pool consisting of high-performing validators. The design focuses on enabling widespread participation and ensuring consistent performance. By combining these pools, ETHx aims to provide a robust and scalable infrastructure for node operations in a decentralized manner.

| Issues Summarize|
|-----------------|

| Risk    | Issues Details                                                                                                |
|---------|---------------------------------------------------------------------------------------------------------------|
| [QA-01] | `MIN_AUCTION_DURATION` will not be accurate due to ethereum average block time volatility                     |
| [QA-02] | `setCommissionFees()` is missing sanity checks                                                                |
| [QA-03] | `WithdrawRequestInfo` is not enforced in the code                                                             |
| [QA-04] | `BURNER_ROLE` can burn any arbitrary amount of ETHx token from any account                                    |
| [QA-05] | Potential temporary griefing attack when `REWARD_THRESHOLD` is lowered                                        |
| [QA-06] | `VaultFactory.deployWithdrawVault()`: deterministic address may cause issues                                  |

### [QA-01] `MIN_AUCTION_DURATION` will not be accurate due to ethereum average block time volatility

#### Impact

The `MIN_AUCTION_DURATION` value may not accurately represent 24 Hours in blocks. The value is currently calculated based on a fixed 12-second Ethereum block time, but Ethereum's average block time can vary over time, causing potential discrepancies between the calculated block count and actual elapsed time.

#### Proof of Concept

[`MIN_AUCTION_DURATION`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L22) is defined as follow:

```solidity
    uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours
```

`MIN_AUCTION_DURATION` should represent `24 hours` in blocks which may differ based on this [site](https://ycharts.com/indicators/ethereum_average_block_time):

> Ethereum Average Block Time is at a current level of 12.13, up from 12.12 yesterday and down from 14.30 one year ago. This is a change of 0.08% from yesterday and -15.17% from one year ago.

To demonstrate this potential Risk, here is 2 scenarios based on the Highest and the Lowest block time value in the past two months:
 
##### Scenario 1: May 12, 2023(Highest block time in past two months)

- 24 hours = 24 x 60 x 60 = 86_400 seconds
- Dividing 86_400 seconds by the block time of 12.45 seconds gives us:
- 86_400 / 12.45 = 6940 Ethereum blocks
- The diffrence in blocks is: 260 Ethereum blocks
- The diffrence in minutes is: 54 minutes


##### Scenario 2: May 28, 2023(Lowest block time in past two months)

- 24 hours = 24 x 60 x 60 = 86_400 seconds
- Dividing 86_400 seconds by the block time of 12.08 seconds gives us:
- 86_400 / 12.08 = 7152 Ethereum blocks
- The diffrence in blocks is: 48 Ethereum blocks
- The diffrence in minutes is: 10 minutes

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

We recommend using the `block.timestamp` and [`Time Units`](https://docs.soliditylang.org/en/v0.8.20/units-and-global-variables.html#time-units) instead of `block.numbe`r to calculate time intervals. This approach is more reliable than using a fixed block count, which can be affected by Ethereum's average block time variability.

### [QA-02] `setCommissionFees()` is missing sanity checks

#### Impact

[`setCommissionFees()`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L66-L75) function is used to set the commission fees for the protocol. However, the current implementation lacks sanity checks, specifically a minimum bound check, which makes it vulnerable to human error and potential loss of funds for either the protocol or the operator.

#### Proof of Concept

```solidity
    function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {
            revert InvalidCommission();
        }
        protocolFee = _protocolFee;
        operatorFee = _operatorFee;

        emit UpdatedCommissionFees(_protocolFee, _operatorFee);
    }
```

- [PermissionlessPool.sol#L66-L75](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L66-L75)

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

To address this issue, introduce a `MIN_COMMISSION_LIMIT_BIPS` constant that represents the minimum allowed commission limit in basis points.

### [QA-03] `WithdrawRequestInfo` is not enforced in the code

#### Impact

The `WithdrawRequestInfo` struct in the code represents a withdrawal delay to prevent instant withdrawal of the operator's SD token. However, the code currently lacks enforcement of this withdrawal delay, and the operator receives the requested amount in the same tx as you can see in this [line](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L68):

```solidity
        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
```

#### Proof of Concept

```solidity
    /// @notice for operator to request withdraw of sd
    /// @dev it does not transfer sd tokens immediately
    /// operator should come back after withdrawal-delay time to claim
    /// this requested sd is subject to slashes
    function withdraw(uint256 _requestedSD) external override {
        address operator = msg.sender;
        uint256 opSDBalance = operatorSDBalance[operator];

        if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
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

- [SDCollateral.sol#L54-L73](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L54-L73)

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

To address this issue, we recommend one of the following :

- Enforce Withdrawal Delay

- Update NatSpec Documentation and remove the `WithdrawRequestInfo` struct from [`ISDCollateral`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/interfaces/SDCollateral/ISDCollateral.sol#L14-L17)

### [QA-04] `BURNER_ROLE` can burn any arbitrary amount of ETHx token from any account   

#### Impact

[`ETHx.burnFrom()`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L56-L58) allow the burner role to burn any arbitrary amount from any arbitrary account which will provides less guarantees and reduces the level of trust required, thus increasing risk for users, and it will make the protocol more centralized which conflicts with what the Stader protocol is aiming to provide:

> ETHx aims to provide stakers, a decentralized and scalable solution with diverse DeFi integrations.

#### Proof of Concept

```solidity
    function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {
        _burn(account, amount);
    }
```

- [ETHx.sol#L56-L58](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L56-L58)

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

We recommend updating the function to: 

```solidity
    function burnFrom(uint256 amount) external whenNotPaused {
        _burn(msg.sender, amount);
    }
```

### [QA-05] Potential temporary griefing attack when `REWARD_THRESHOLD` is lowered 

#### Impact

There is a potential temporary griefing attack when the `REWARD_THRESHOLD` is set or updated to a lower value. This attack can cause the [`distributeRewards()`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L30-L52) function to revert due to the following check:

```solidity
        if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
            emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
            revert InvalidRewardAmount();
        }
```

The purpose of this check is to distinguish whether the ETH balance belongs to rewards or staked ETH as per of the sponsor:

> Both consensus layer rewards and "32 bonded ETH" is sent back to the same address (validatorWithdrawalVault). So it becomes important to distinguish whether ETH balance belongs to rewards or staked ETH (that's now withdrawn)
We use rewards threshold to make that distinction.

However, this check makes the contract vulnerable to temporary griefing attacks. An attacker can exploit this vulnerability by sending ETH to the contract and causing `totalRewards > staderConfig.getRewardsThreshold()` to be true. The likelihood of the attack depends on the value of `REWARD_THRESHOLD` — it is more expensive when `REWARD_THRESHOLD` is high and cheaper when `REWARD_THRESHOLD` is low, thus introducing a `MIN_LIMIT_REWARD_THRESHOLD` will make the contract more robust.

#### Proof of Concept

```solidity
    function distributeRewards() external override {
        uint8 poolId = VaultProxy(payable(address(this))).poolId();
        uint256 validatorId = VaultProxy(payable(address(this))).id();
        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
        uint256 totalRewards = address(this).balance;
        if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {
            emit DistributeRewardFailed(totalRewards, staderConfig.getRewardsThreshold());
            revert InvalidRewardAmount();
        }
        if (totalRewards == 0) {
            revert NotEnoughRewardToDistribute();
        }
        (uint256 userShare, uint256 operatorShare, uint256 protocolShare) = IPoolUtils(staderConfig.getPoolUtils())
            .calculateRewardShare(poolId, totalRewards);

        // Distribute rewards
        IStaderStakePoolManager(staderConfig.getStakePoolManager()).receiveWithdrawVaultUserShare{value: userShare}();
        UtilLib.sendValue(payable(staderConfig.getStaderTreasury()), protocolShare);
        IOperatorRewardsCollector(staderConfig.getOperatorRewardsCollector()).depositFor{value: operatorShare}(
            getOperatorAddress(poolId, validatorId, staderConfig)
        );
        emit DistributedRewards(userShare, operatorShare, protocolShare);
    }
```

- [ValidatorWithdrawalVault.sol#L30-L52](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L30-L52)

```solidity
    function updateRewardsThreshold(uint256 _rewardsThreshold) external onlyRole(MANAGER) {
        setVariable(REWARD_THRESHOLD, _rewardsThreshold);
    }
```

- [StaderConfig.sol#L118-L120](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L118-L120)

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

We recommend introducing a `MIN_LIMIT_REWARD_THRESHOLD` and check that the new `REWARD_THRESHOLD` is not below the `MIN_LIMIT_REWARD_THRESHOLD` to protect against Dos, griefing attacks in the furture.

### [QA-06] `VaultFactory.deployWithdrawVault()`: deterministic address may cause issues 

#### Impact

The `VaultFactory.deployWithdrawVault()` function is currently using the `lonesUpgradeable.cloneDeterministic` function to create a new withdrawal vault. However, the address of the newly created vault depends on the `vaultProxyImplementation` and the `salt` parameter provided. Once the tx is broadcasted, the `salt` parameter can be viewed by anyone watching the public mempool and the public readability of `salt` parameter may cause issues in the future.

#### Proof of Concept

```solidity
    function deployNodeELRewardVault(uint8 _poolId, uint256 _operatorId)
        public
        override
        onlyRole(NODE_REGISTRY_CONTRACT)
        returns (address)
    {
        bytes32 salt = sha256(abi.encode(_poolId, _operatorId));
        address nodeELRewardVaultAddress = ClonesUpgradeable.cloneDeterministic(vaultProxyImplementation, salt);
        VaultProxy(payable(nodeELRewardVaultAddress)).initialise(false, _poolId, _operatorId, address(staderConfig));

        emit NodeELRewardVaultCreated(nodeELRewardVaultAddress);
        return nodeELRewardVaultAddress;
    }
```

- [VaultFactory.sol#L48C2-L60](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L48C2-L60)

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

We recommend including the `msg.sender` address as part of the salt hash:

```solidity
bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount, msg.sender));
```
