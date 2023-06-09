
## Low Risk Issues
|      |Issue  | Instances |
|:----:|:-----:|:---------:|
| L-01 |Loss of precision|     9     |
| L-02 |Setters should have initial value check|    10     |
| L-03 |Use of floating pragma|     1     |

## Not Critical Issues
|       |Issue  | Instances |
|:-----:|:-----:|:---------:|
| NC-01 |Use scientific notation (e.g. ```1e18```) rather than exponentiation (e.g. ```10**18```)|     6     |
| NC-02 |Typos|     1     |
| NC-03 |Variable names don't follow the Solidity style guide|     1     |
| NC-04 |```public``` functions not called by the contract should be declared ```external``` instead|    12     |
| NC-05 |Let's return default value instead of explicity write it|    1     |

# Low Risk Issues
## L-01
### Loss of precision
#### Description
Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator


<details>
<summary>There are 9 instances of this issue not reported by bot race:</summary>

```
File: PermissionedNodeRegistry.sol

  
  201         uint256 validatorPerOperator = _numValidators / totalActiveOperatorCount;

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L201](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L201)


```
File: PermissionlessPool.sol

  
  128         uint256 requiredValidators = msg.value / (staderConfig.getFullDepositSize() - DEPOSIT_NODE_BOND);

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L128](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L128)

```
File: PoolSelector.sol

  
  61         uint256 poolTotalTarget = (poolWeights[_poolId] * totalValidatorsRequired) / POOL_WEIGHTS_SUM;

  
  93             uint256 remainingValidatorsToDeposit = ethToDeposit / poolDepositSize;

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L61](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L61)

```
File: PoolUtils.sol

  
  263         protocolShare = (protocolFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();

  
  265         operatorShare = (_totalRewards * collateralETH) / TOTAL_STAKED_ETH;

  
  266         operatorShare += (operatorFeeBps * _userShareBeforeCommision) / staderConfig.getTotalFee();

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L261](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L261)


```
File: StaderOracle.sol

  
  603         return (block.number / updateFrequency) * updateFrequency;

  
  665             !(newExchangeRate >= (currentExchangeRate * (ER_CHANGE_MAX_BPS - erChangeLimit)) / ER_CHANGE_MAX_BPS &&

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L603](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L603)

            
</details>

## L-02
### Setters should have initial value check
#### Description
Setters should have initial value check to prevent assigning wrong value to the variable. Assginment of wrong value can lead to unexpected behavior of the contract.


<details>
<summary>There are 10 instances of this issue:</summary>

```
File: Penalty.sol

  @audit: unchecked values _pubkey, _amount
  53     function setAdditionalPenaltyAmount(bytes calldata _pubkey, uint256 _amount) external override {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L53](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L53)

```
File: StaderConfig.sol

  @audit: unchecked values key, val
  471     function setConstant(bytes32 key, uint256 val) internal {

  @audit: unchecked values key, val
  476     function setVariable(bytes32 key, uint256 val) internal {

  @audit: unchecked values key, val
  481     function setAccount(bytes32 key, address val) internal {

  @audit: unchecked values key, val
  487     function setContract(bytes32 key, address val) internal {

  @audit: unchecked values key, val
  493     function setToken(bytes32 key, address val) internal {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L471](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L471)

```
File: StaderOracle.sol

  @audit: unchecked values _updateFrequency
  539     function setSDPriceUpdateFrequency(uint256 _updateFrequency) external override {

  @audit: unchecked values _updateFrequency
  544     function setValidatorStatsUpdateFrequency(uint256 _updateFrequency) external override {

  @audit: unchecked values _updateFrequency
  549     function setWithdrawnValidatorsUpdateFrequency(uint256 _updateFrequency) external override {

  @audit: unchecked values _updateFrequency
  554     function setMissedAttestationPenaltyUpdateFrequency(uint256 _updateFrequency) external override {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L539](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L539)

            
</details>

## L-03
### Use of floating pragma
#### Description
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.[(https://swcregistry.io/docs/SWC-103)](https://swcregistry.io/docs/SWC-103)


```
File: VaultProxy.sol

  
  2 pragma solidity ^0.8.16;

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L2](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L2)


# Not Critical Issues

## NC-01
### Use scientific notation (e.g. ```1e18```) rather than exponentiation (e.g. ```10**18```)
#### Description
While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist


<details>
<summary>There are 6 instances of this issue not reported by bot race:</summary>

```
File: PoolSelector.sol

  
  21     uint256 public constant POOL_WEIGHTS_SUM = 10000;

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L21](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L21)

```
File: StaderConfig.sol

  92         setConstant(TOTAL_FEE, 10000);

  
  96         setVariable(MAX_DEPOSIT_AMOUNT, 10000 ether);

  
  98         setVariable(MAX_WITHDRAW_AMOUNT, 10000 ether);

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L93](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L93)

```
File: StaderOracle.sol

  
  27     uint256 public constant ER_CHANGE_MAX_BPS = 10000;

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L27](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L27)

```
File: UserWithdrawalManager.sol

  
  64         maxNonRedeemedUserRequestCount = 1000;

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L64](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L64)

            
</details>

## NC-02
### Typos
#### Description



```
File: VaultProxy.sol

  @audit: contrat/contract
  63     /**
  64      * @notice @update the owner of vault proxy contrat
  65      * @dev only owner can call
  66      * @param _owner new owner account
  67      */

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L63-L67](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L63-L67)


## NC-03
### Variable names don't follow the Solidity style guide
#### Description
For ```constant``` variable names, each word should use all capital letters, with underscores separating each word (CONSTANT_CASE)


```
File: StaderConfig.sol

  
  72     bytes32 public constant ETHx = keccak256('ETHx');

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L72](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L72)

## NC-04
### ```public``` functions not called by the contract should be declared ```external``` instead
#### Description
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from ```external``` to ```public```.


<details>
<summary>There are 12 instances of this issue not reported by bot race:</summary>

```
File: ETHx.sol

  
  29     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L29](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L29)

```
File: PermissionedNodeRegistry.sol

  
  66     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L66](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L66)

```
File: PermissionedPool.sol

  
  40     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L40](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L40)

```
File: PermissionlessNodeRegistry.sol

  
  66     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L66](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L66)

```
File: PermissionlessPool.sol

  
  38     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L38](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L38)

```
File: PoolUtils.sol

  
  25     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L25](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L25)

```
File: SDCollateral.sol

  
  26     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L26](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L26)

```
File: SocializingPool.sol

  
  39     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L39](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L39)

```
File: StaderInsuranceFund.sol

  
  26     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L26](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L26)

```
File: StaderOracle.sol

  
  62     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L62](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L62)

```
File: StaderStakePoolsManager.sol

  
  50     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L50](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L50)

```
File: UserWithdrawalManager.sol

  
  54     function initialize(address _admin, address _staderConfig) public initializer {

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L54](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L54)

            
</details>


## NC-05
### Let's return default value instead of explicity write it
#### Description

Let empty body and consider to comment why.

```diff
File: PermissionedNodeRegistry.sol

  
  546     function getCollateralETH() external pure override returns (uint256) {
- 547       return 0;
  548     }

```
[https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L546-L548](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L546-L548)
