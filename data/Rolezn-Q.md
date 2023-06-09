## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | IERC20 `approve()` Is Deprecated | 1 |
| [LOW&#x2011;2](#LOW&#x2011;2) | Approve `type(uint256).max` not work with some tokens | 1 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Array lengths not checked | 7 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Consider the case where `totalETHXSupply` is 0 | 1 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Do not allow fees to be set to `100%` | 2 |
| [LOW&#x2011;6](#LOW&#x2011;6) | Lack of checks-effects-interactions | 2 |
| [LOW&#x2011;7](#LOW&#x2011;7) | Minting tokens to the zero address should be avoided | 1 |
| [LOW&#x2011;8](#LOW&#x2011;8) | Missing Contract-existence Checks Before Low-level Calls | 8 |
| [LOW&#x2011;9](#LOW&#x2011;9) | Missing Checks for Address(0x0)  | 6 |
| [LOW&#x2011;10](#LOW&#x2011;10) | Contracts are not using their OZ Upgradeable counterparts | 13 |
| [LOW&#x2011;11](#LOW&#x2011;11) | Contracts are designed to receive ETH but do not implement function for withdrawal | 6 |
| [LOW&#x2011;12](#LOW&#x2011;12) | Unbounded loop | 5 |
| [LOW&#x2011;13](#LOW&#x2011;13) | No Storage Gap For Upgradeable Contracts | 18 |
| [LOW&#x2011;14](#LOW&#x2011;14) | Upgrade OpenZeppelin Contract Dependency | 2 |
| [LOW&#x2011;15](#LOW&#x2011;15) | Variables are hardcoded based on a 7200 blocks per day basis | 1 |
| [LOW&#x2011;16](#LOW&#x2011;16) | Minimum auction duration is unclear | 1 |
| [LOW&#x2011;17](#LOW&#x2011;17) | `depositFor` should also have a `whenNotPaused` modifier | 1 |


Total: 76 contexts over 17 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Critical Changes Should Use Two-step Procedure | 16 |
| [NC&#x2011;2](#NC&#x2011;2) | `block.timestamp` is already used when emitting events, no need to input timestamp | 1 |
| [NC&#x2011;3](#NC&#x2011;3) | Event emit should emit a parameter | 2 |
| [NC&#x2011;4](#NC&#x2011;4) | No need to initialize uints to zero | 3 |
| [NC&#x2011;5](#NC&#x2011;5) | Initial value check is missing in Set Functions | 14 |
| [NC&#x2011;6](#NC&#x2011;6) | Missing event for critical parameter change | 5 |
| [NC&#x2011;7](#NC&#x2011;7) | Public Functions Not Called By The Contract Should Be Declared External Instead | 15 |
| [NC&#x2011;8](#NC&#x2011;8) | Empty blocks should be removed or emit something | 3 |
| [NC&#x2011;9](#NC&#x2011;9) | Redundant Cast | 1 |
| [NC&#x2011;10](#NC&#x2011;10) | Large multiples of ten should use scientific notation | 5 |
| [NC&#x2011;11](#NC&#x2011;11) | `calculateRewardShare` will revert when `TOTAL_STAKED_ETH` is 0 | 1 |

Total: 66 contexts over 11 issues

## Low Risk Issues

### <a href="#QA Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> IERC20 `approve()` Is Deprecated

`approve()` is subject to a known front-running attack. It is deprecated in favor of `safeIncreaseAllowance()` and `safeDecreaseAllowance()`. If only setting the initial allowance to the value that means infinite, `safeIncreaseAllowance()` can be used instead.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#IERC20-approve-address-uint256-

#### <ins>Proof Of Concept</ins>

<details>

```solidity
106: IERC20(staderConfig.getStaderToken()).approve(auctionContract, type(uint256).max);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SDCollateral.sol#L106



</details>

#### <ins>Recommended Mitigation Steps</ins>

Consider using `safeIncreaseAllowance()` / `safeDecreaseAllowance()` instead.


### <a href="#QA Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Array lengths not checked

If the length of the arrays are not required to be of the same length, user operations may not be fully executed

#### <ins>Proof Of Concept</ins>


<details>

```solidity

function addValidatorKeys(
        bytes[] calldata _pubkey,
        bytes[] calldata _preDepositSignature,
        bytes[] calldata _depositSignature
    ) external override whenNotPaused {

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L140

```solidity

function markValidatorReadyToDeposit(
        bytes[] calldata _readyToDepositPubkey,
        bytes[] calldata _frontRunPubkey,
        bytes[] calldata _invalidSignaturePubkey
    ) external override nonReentrant whenNotPaused {

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L254

```solidity

function addValidatorKeys(
        bytes[] calldata _pubkey,
        bytes[] calldata _preDepositSignature,
        bytes[] calldata _depositSignature
    ) external payable override nonReentrant whenNotPaused {

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L125

```solidity

function markValidatorReadyToDeposit(
        bytes[] calldata _readyToDepositPubkey,
        bytes[] calldata _frontRunPubkey,
        bytes[] calldata _invalidSignaturePubkey
    ) external override nonReentrant whenNotPaused {

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L183

```solidity

function preDepositOnBeaconChain(
        bytes[] calldata _pubkey,
        bytes[] calldata _preDepositSignature,
        uint256 _operatorId,
        uint256 _operatorTotalKeys
    ) external payable nonReentrant {

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessPool.sol#L85

```solidity

function claim(
        uint256[] calldata _index,
        uint256[] calldata _amountSD,
        uint256[] calldata _amountETH,
        bytes32[][] calldata _merkleProof
    ) external override nonReentrant whenNotPaused {

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SocializingPool.sol#L107

```solidity

function _claim(
        uint256[] calldata _index,
        address _operator,
        uint256[] calldata _amountSD,
        uint256[] calldata _amountETH,
        bytes32[][] calldata _merkleProof
    ) internal returns (uint256 _totalAmountSD, uint256 _totalAmountETH) {

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SocializingPool.sol#L137



</details>




### <a href="#QA Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Consider the case where `totalETHXSupply` is 0

Consider the case where `totalETHXSupply` is 0. When `totalETHXSupply` is 0, it should return 0 directly, because there will be an error of dividing by 0.

#### <ins>Impact</ins>

This would cause the affected functions to revert and as a result can lead to potential loss.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
163: : (totalETHBalance * DECIMALS) / totalETHXSupply;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/library/UtilLib.sol#L163



</details>


#### <ins>Recommended Mitigation Steps</ins>
Add check for zero value and return 0.

```solidity
if ( totalETHXSupply == 0) return 0;
```



### <a href="#QA Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Do not allow fees to be set to `100%`

It is recommended from a risk perspective to disallow setting 100% fees at all, which is when `_protocolFee + _operatorFee == MAX_COMMISSION_LIMIT_BIPS`. A reasonable fee maximum should be checked for instead or also check when it is equal to `MAX_COMMISSION_LIMIT_BIPS`, i.e. `if (_protocolFee + _operatorFee >= MAX_COMMISSION_LIMIT_BIPS)`

#### <ins>Proof Of Concept</ins>


<details>

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

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedPool.sol#L236

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

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessPool.sol#L66



</details>






### <a href="#QA Summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Lack of checks-effects-interactions

It's recommended to execute external calls after state changes, to prevent reentrancy bugs.

Consider moving the external calls after the state changes on the following instances.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
47: function depositSDAsCollateral(uint256 _sdAmount) external override {
        address operator = msg.sender;
        operatorSDBalance[operator] += _sdAmount;

        if (!IERC20(staderConfig.getStaderToken()).transferFrom(operator, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }

        emit SDDeposited(operator, _sdAmount);
    }

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SDCollateral.sol#L47

```solidity
68: function withdraw(uint256 _requestedSD) external override {
        address operator = msg.sender;
        uint256 opSDBalance = operatorSDBalance[operator];

        if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
            revert InsufficientSDToWithdraw(opSDBalance);
        }
        operatorSDBalance[operator] -= _requestedSD;

        
        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
            revert SDTransferFailed();
        }

        emit SDWithdrawn(operator, _requestedSD);
    }

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SDCollateral.sol#L68



</details>






### <a href="#QA Summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Minting tokens to the zero address should be avoided

The core function `mint` is used by users to mint an option position by providing token1 as collateral and borrowing the max amount of liquidity. `Address(0)` check is missing in both this function and the internal function `_mint`, which is triggered to mint the tokens to the to address. Consider applying a check in the function to ensure tokens aren't minted to the zero address.

#### <ins>Proof Of Concept</ins>


<details>

```solidity

function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) whenNotPaused {
        _mint(to, amount);
    }

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/ETHx.sol#L47



</details>






### <a href="#QA Summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> Missing Contract-existence Checks Before Low-level Calls

Low-level calls return success if there is no code present at the specified address. 

#### <ins>Proof Of Concept</ins>


<details>

```solidity
131: (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Auction.sol#L131

```solidity
91: (bool success, ) = payable(staderConfig.getStaderTreasury()).call{value: _rewardsData.protocolETHRewards}('');
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SocializingPool.sol#L91

```solidity
121: (success, ) = payable(operatorRewardsAddr).call{value: totalAmountETH}('');
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SocializingPool.sol#L121

```solidity
48: (bool success, ) = payable(msg.sender).call{value: _amount}('');
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderInsuranceFund.sol#L48

```solidity
102: (bool success, ) = payable(staderConfig.getUserWithdrawManager()).call{value: _amount}('');
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderStakePoolsManager.sol#L102

```solidity
229: (bool success, ) = _recipient.call{value: _amount}('');
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/UserWithdrawalManager.sol#L229

```solidity
45: (bool success, bytes memory data) = vaultImplementation.delegatecall(_input);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/VaultProxy.sol#L45

```solidity
168: (bool success, ) = payable(_receiver).call{value: _amount}('');
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/library/UtilLib.sol#L168



</details>


#### <ins>Recommended Mitigation Steps</ins>

In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0`



### <a href="#QA Summary">[LOW&#x2011;10]</a><a name="LOW&#x2011;10"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

<details>

```solidity
9: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Auction.sol#L9

```solidity
16: import '@openzeppelin/contracts/utils/math/Math.sol';

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L16

```solidity
17: import '@openzeppelin/contracts/utils/math/Math.sol';

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedPool.sol#L17

```solidity
15: import '@openzeppelin/contracts/utils/math/Math.sol';

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessPool.sol#L15

```solidity
10: import '@openzeppelin/contracts/utils/math/Math.sol';
11: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PoolSelector.sol#L10-L11

```solidity
11: import '@openzeppelin/contracts/utils/math/Math.sol';
14: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SDCollateral.sol#L11-L14

```solidity
15: import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SocializingPool.sol#L15

```solidity
16: import '@openzeppelin/contracts/utils/math/Math.sol';
17: import '@openzeppelin/contracts/utils/math/SafeMath.sol';

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderStakePoolsManager.sol#L16-L17

```solidity
12: import '@openzeppelin/contracts/utils/math/Math.sol';

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/UserWithdrawalManager.sol#L12

```solidity
16: import '@openzeppelin/contracts/utils/math/Math.sol';

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/ValidatorWithdrawalVault.sol#L16



</details>

#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.
See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for list of available upgradeable contracts




### <a href="#QA Summary">[LOW&#x2011;11]</a><a name="LOW&#x2011;11"> Contracts are designed to receive ETH but do not implement function for withdrawal

The following contracts can receive ETH but can not withdraw, ETH is occasionally sent by users will be stuck in those contracts.
This functionality also applies to baseTokens resulting in locked tokens and loss of funds.


#### <ins>Proof Of Concept</ins>

<details>

```solidity
52: receive() external payable {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedPool.sol#L52

```solidity
50: receive() external payable {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessPool.sol#L50

```solidity
57: receive() external payable {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SocializingPool.sol#L57

```solidity
68: receive() external payable {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderStakePoolsManager.sol#L68

```solidity
68: receive() external payable {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/UserWithdrawalManager.sol#L68

```solidity
26: receive() external payable {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/ValidatorWithdrawalVault.sol#L26



</details>

#### <ins>Recommended Mitigation Steps</ins>
Provide a rescue ETH and rescueTokens function



### <a href="#QA Summary">[LOW&#x2011;12]</a><a name="LOW&#x2011;12"> Unbounded loop

New items are pushed into the following arrays but there is no option to `pop` them out. Currently, the array can grow indefinitely. E.g. there's no maximum limit and there's no functionality to remove array values.
If the array grows too large, calling relevant functions might run out of gas and revert. Calling these functions could result in a DOS condition.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
174: validatorIdsByOperatorId[operatorId].push(nextValidatorId);

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L174

```solidity
160: validatorIdsByOperatorId[operatorId].push(nextValidatorId);

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L160

```solidity
44: poolIdArray.push(_poolId);

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PoolUtils.sol#L44

```solidity
301: sdPrices.push(_sdPrice);

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L301

```solidity
107: requestIdsByUserAddress[_owner].push(nextRequestId);

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/UserWithdrawalManager.sol#L107



</details>

#### <ins>Recommended Mitigation Steps</ins>
Add a functionality to delete array values or add a maximum size limit for arrays.






### <a href="#QA Summary">[LOW&#x2011;13]</a><a name="LOW&#x2011;13"> No storage gap for upgradeable contract might lead to storage slot collision

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts.

Refer to the bottom part of this article: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

#### <ins>Proof Of Concept</ins>

However, the contract doesn't contain a storage gap. The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

<details>

```solidity
Auction.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Auction.sol#L10

```solidity
ETHx.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/ETHx.sol#L7

```solidity
OperatorRewardsCollector.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/OperatorRewardsCollector.sol#L9

```solidity
Penalty.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Penalty.sol#L11

```solidity
PermissionedNodeRegistry.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L17

```solidity
PermissionedPool.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedPool.sol#L18

```solidity
PermissionlessNodeRegistry.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L19

```solidity
PermissionlessPool.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessPool.sol#L16

```solidity
PoolSelector.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PoolSelector.sol#L12

```solidity
PoolUtils.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PoolUtils.sol#L10

```solidity
SDCollateral.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SDCollateral.sol#L12

```solidity
SocializingPool.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SocializingPool.sol#L11

```solidity
StaderConfig.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L8

```solidity
StaderInsuranceFund.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderInsuranceFund.sol#L10

```solidity
StaderOracle.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L13

```solidity
StaderStakePoolsManager.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderStakePoolsManager.sol#L18

```solidity
UserWithdrawalManager.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/UserWithdrawalManager.sol#L13

```solidity
VaultFactory.sol
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/factory/VaultFactory.sol#L9



</details>

#### <ins>Recommended Mitigation Steps</ins>
Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. Please reference OpenZeppelin upgradeable contract templates.
```solidity
    uint256[50] private __gap;
```



### <a href="#QA Summary">[LOW&#x2011;14]</a><a name="LOW&#x2011;14"> Upgrade OpenZeppelin Contract Dependency

An outdated OZ version is used (which has known vulnerabilities, see: https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories).
Release: https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0

#### <ins>Proof Of Concept</ins>

Require dependencies to be at least version of 4.9.0 to include critical vulnerability patches.

<details>

```solidity
    "@openzeppelin/contracts": "^4.7.3"
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/package.json#L37

```solidity
    "@openzeppelin/contracts-upgradeable": "^4.8.0-rc.1"
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/package.json#L38



</details>

#### <ins>Recommended Mitigation Steps</ins>

Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.9.0 via `>=4.9.0`



### <a href="#QA Summary">[LOW&#x2011;15]</a><a name="LOW&#x2011;15"> Variables are hardcoded based on a 7200 blocks per day basis

The following variables are hardcoded and assume that the number of blocks per day is 7200 and the number of seconds per block is 12. Yet, it is possible that the number of seconds per block is more or less than 12 due to network traffic and future chain upgrades. When the number of seconds per block is no longer 12, the durations corresponding to the variables will no longer in the clean ration of 7200 blocks, which break the duration specifications for various phases. 
To avoid unexpectedness and disputes, please consider using `block.timestamp` instead of blocks for defining durations for various phases.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
22: uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Auction.sol#L22



</details>



### <a href="#QA Summary">[LOW&#x2011;16]</a><a name="LOW&#x2011;16"> Minimum auction duration is unclear

It is currently unclear if the minimum auction duration is either `MIN_AUCTION_DURATION ` which is 24 hours or `2 * MIN_AUCTION_DURATION`.
As in the `initialize` function you can see that the auction duration is set to `2 * MIN_AUCTION_DURATION` which is 48 hours.
However, in the function `updateDuration` you will see the limit to `if (_duration < MIN_AUCTION_DURATION) revert ShortDuration();` which is require a minimum duration of only 24 hours compared to 48 hours.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
22: uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Auction.sol#L22

```solidity
38: duration = 2 * MIN_AUCTION_DURATION;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L38

```solidity
146: if (_duration < MIN_AUCTION_DURATION) revert ShortDuration();
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L146


### <a href="#QA Summary">[LOW&#x2011;17]</a><a name="LOW&#x2011;17"> `depositFor` should also have a `whenNotPaused` modifier

When the protocol is paused, it is no longer possible to call the `claim()` function. It should also make sense to disallow users from depositing using the `depositFor` function when the protocol is paused to avoid any locked funds during the pause.

```solidity
    function depositFor(address _receiver) external payable {
        balances[_receiver] += msg.value;

        emit DepositedFor(msg.sender, _receiver, msg.value);
    }

    function claim() external whenNotPaused {
        address operator = msg.sender;
        uint256 amount = balances[operator];
        balances[operator] -= amount;

        address operatorRewardsAddr = UtilLib.getOperatorRewardAddress(msg.sender, staderConfig);
        UtilLib.sendValue(operatorRewardsAddr, amount);
        emit Claimed(operatorRewardsAddr, amount);
    }
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L40-L54

## Non Critical Issues



### <a href="#QA Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Avoid the use of sensitive terms

Use <a href="https://www.zdnet.com/article/mysql-drops-master-slave-and-blacklist-whitelist-terminology/">alternative variants</a>, e.g. allowlist/denylist instead of whitelist/blacklist

#### <ins>Proof Of Concept</ins>

<details>

```solidity
53: // mapping of whitelisted permissioned node operator
85: * @dev only `MANAGER` can call, whitelisting a one way change there is no blacklisting
88: function whitelistPermissionedNOs(address[] calldata _permissionedNOs) external override {
95: emit OperatorWhitelisted(operator);
101: * @dev only whitelisted NOs can call

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L53

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L85

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L88

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L95

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L101





</details>



### <a href="#QA Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Variable Names That Consist Of All Capital Letters Should Be Reserved For Const/immutable Variables

If the variable needs to be different based on which class it comes from, a view/pure function should be used instead.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
125: uint256 DECIMALS = staderConfig.getDecimals();
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/UserWithdrawalManager.sol#L125

```solidity
160: uint256 DECIMALS = _staderConfig.getDecimals();
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/library/UtilLib.sol#L160



</details>




### <a href="#QA Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Consider using named mappings

Consider moving to solidity version 0.8.18 or later, and using <a href='https://ethereum.stackexchange.com/a/145555'>named mappings</a> to make it easier to understand the purpose of each mapping

#### <ins>Proof Of Concept</ins>

<details>

```solidity
20: mapping(uint256 => LotItem) public lots;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Auction.sol#L20

```solidity
20: mapping(address => uint256) public balances;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/OperatorRewardsCollector.sol#L20

```solidity
22: mapping(bytes32 => uint256) public override additionalPenaltyAmount;
24: mapping(bytes => uint256) public override totalPenaltyAmount;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Penalty.sol#L22

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Penalty.sol#L24



```solidity
46: mapping(uint256 => Validator) public override validatorRegistry;
48: mapping(bytes => uint256) public override validatorIdByPubkey;
50: mapping(uint256 => Operator) public override operatorStructById;
52: mapping(address => uint256) public override operatorIDByAddress;
54: mapping(address => bool) public override permissionList;
56: mapping(uint256 => uint256[]) public override validatorIdsByOperatorId;
58: mapping(uint256 => uint256) public override nextQueuedValidatorIndexByOperatorId;
59: mapping(uint256 => uint256) public socializingPoolStateChangeBlock;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L46

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L48

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L50

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L52

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L54

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L56

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L58

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L59



```solidity
46: mapping(uint256 => Validator) public override validatorRegistry;
48: mapping(bytes => uint256) public override validatorIdByPubkey;
50: mapping(uint256 => uint256) public override queuedValidators;
52: mapping(uint256 => Operator) public override operatorStructById;
54: mapping(address => uint256) public override operatorIDByAddress;
56: mapping(uint256 => uint256[]) public override validatorIdsByOperatorId;
57: mapping(uint256 => uint256) public socializingPoolStateChangeBlock;
59: mapping(uint256 => address) public override nodeELRewardVaultByOperatorId;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L46

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L48

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L50

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L52

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L54

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L56

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L57

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L59



```solidity
23: mapping(uint8 => uint256) public poolWeights;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PoolSelector.sol#L23

```solidity
17: mapping(uint8 => address) public override poolAddressById;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PoolUtils.sol#L17

```solidity
18: mapping(uint8 => PoolThresholdInfo) public poolThresholdbyPoolId;
19: mapping(address => uint256) public override operatorSDBalance;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SDCollateral.sol#L18-L19

```solidity
29: mapping(address => mapping(uint256 => bool)) public override claimedRewards;
30: mapping(uint256 => bool) public handledRewards;
32: mapping(uint256 => RewardsData) public rewardsDataMap;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SocializingPool.sol#L29

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SocializingPool.sol#L30

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SocializingPool.sol#L32



```solidity
74: mapping(bytes32 => uint256) private constantsMap;
75: mapping(bytes32 => uint256) private variablesMap;
76: mapping(bytes32 => address) private accountsMap;
77: mapping(bytes32 => address) private contractsMap;
78: mapping(bytes32 => address) private tokensMap;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L74-L78

```solidity
44: mapping(address => bool) public override isTrustedNode;
45: mapping(bytes32 => bool) private nodeSubmissionKeys;
46: mapping(bytes32 => uint8) private submissionCountKeys;
47: mapping(bytes32 => uint16) public override missedAttestationPenalty;
55: mapping(bytes32 => uint256) public updateFrequencyMap;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L44

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L45

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L46

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L47

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L55



```solidity
36: mapping(uint256 => UserWithdrawInfo) public override userWithdrawRequests;
38: mapping(address => uint256[]) public override requestIdsByUserAddress;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/UserWithdrawalManager.sol#L36

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/UserWithdrawalManager.sol#L38





</details>



### <a href="#QA Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Constant redefined elsewhere

Consider defining in only one contract so that values cannot become out of sync when only one location is updated. A <a href="https://medium.com/coinmonks/gas-cost-of-solidity-library-functions-dbe0cedd4678">cheap way</a> to store constants in a single location is to create an `internal constant` in a `library`. If the variable is a local cache of another contract's value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don't get out of sync.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
31: uint8 public constant override POOL_ID = 2;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L31

```solidity
33: uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedPool.sol#L33

```solidity
30: uint8 public constant override POOL_ID = 1;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L30

```solidity
31: uint256 public constant MAX_COMMISSION_LIMIT_BIPS = 1500;

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessPool.sol#L31



</details>



### <a href="#QA Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Constants Should Be Defined Rather Than Using Magic Numbers

Even <a href="https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39">assembly</a> can benefit from using readable constants instead of hex/numeric literals

#### <ins>Proof Of Concept</ins>

<details>

```solidity
92: setConstant(TOTAL_FEE, 10000);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L92

```solidity
94: setConstant(OPERATOR_MAX_NAME_LENGTH, 255);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L94

```solidity
99: setVariable(WITHDRAWN_KEYS_BATCH_SIZE, 50);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L99

```solidity
100: setVariable(MIN_BLOCK_DELAY_TO_FINALIZE_WITHDRAW_REQUEST, 600);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L100

```solidity
70: setUpdateFrequency(ETHX_ER_UF, 7200);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L70

```solidity
71: setUpdateFrequency(SD_PRICE_UF, 7200);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L71

```solidity
72: setUpdateFrequency(VALIDATOR_STATS_UF, 7200);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L72

```solidity
73: setUpdateFrequency(WITHDRAWN_VALIDATORS_UF, 14400);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L73

```solidity
74: setUpdateFrequency(MISSED_ATTESTATION_PENALTY_UF, 50400);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L74



</details>



### <a href="#QA Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Constants in comparisons should appear on the left side

Doing so will prevent <a href="https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html">typo bugs</a>

#### <ins>Proof Of Concept</ins>

<details>

```solidity
98: if (ethAmount == 0) revert NoBidPlaced();
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Auction.sol#L98

```solidity
110: if (lotItem.sdAmount == 0) revert AlreadyClaimed();
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Auction.sol#L110

```solidity
126: if (withdrawalAmount == 0) revert InSufficientETH();
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Auction.sol#L126

```solidity
29: if (totalRewards == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/NodeELRewardVault.sol#L29

```solidity
237: if (remainingValidatorsToDeposit == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L237

```solidity
631: if (operatorId == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L631

```solidity
697: if (keyCount == 0 || keyCount > inputKeyCountLimit) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L697

```solidity
722: if (_operatorId == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedNodeRegistry.sol#L722

```solidity
102: if (validatorToDeposit == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedPool.sol#L102

```solidity
537: if (operatorId == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L537

```solidity
653: if (keyCount == 0 || keyCount > inputKeyCountLimit) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L653

```solidity
682: if (_operatorId == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L682

```solidity
93: if (sdSlashed == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SDCollateral.sol#L93

```solidity
600: if (updateFrequency == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L600

```solidity
202: if (selectedPoolCapacity == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderStakePoolsManager.sol#L202

```solidity
234: if (validatorToDeposit == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderStakePoolsManager.sol#L234

```solidity
39: if (totalRewards == 0) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/ValidatorWithdrawalVault.sol#L39



</details>




### <a href="#QA Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Contract does not follow the Solidity style guide's suggested layout ordering

The <a href="https://docs.soliditylang.org/en/v0.8.20/style-guide.html#order-of-layout">style guide</a> says that, within a contract, the ordering should be 1) Type declarations, 2) State variables, 3) Events, 4) Errors 5) Modifiers, and 6) Functions, but the contract(s) below do not follow this ordering

#### <ins>Proof Of Concept</ins>

<details>

```solidity
ETHx.sol
```





</details>



### <a href="#QA Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Critical Changes Should Use Two-step Procedure

The critical procedures should be two step process.

See similar findings in previous Code4rena contests for reference:
https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure

#### <ins>Proof Of Concept</ins>

<details>

```solidity
53: function setAdditionalPenaltyAmount(bytes calldata _pubkey, uint256 _amount) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Penalty.sol#L53

```solidity
236: function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedPool.sol#L236

```solidity
287: function changeSocializingPoolState(bool _optInForSocializingPool)
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L287

```solidity
66: function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessPool.sol#L66

```solidity
471: function setConstant(bytes32 key, uint256 val) internal {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L471

```solidity
476: function setVariable(bytes32 key, uint256 val) internal {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L476

```solidity
481: function setAccount(bytes32 key, address val) internal {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L481

```solidity
487: function setContract(bytes32 key, address val) internal {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L487

```solidity
493: function setToken(bytes32 key, address val) internal {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L493

```solidity
515: function setERUpdateFrequency(uint256 _updateFrequency) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L515

```solidity
539: function setSDPriceUpdateFrequency(uint256 _updateFrequency) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L539

```solidity
544: function setValidatorStatsUpdateFrequency(uint256 _updateFrequency) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L544

```solidity
549: function setWithdrawnValidatorsUpdateFrequency(uint256 _updateFrequency) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L549

```solidity
554: function setMissedAttestationPenaltyUpdateFrequency(uint256 _updateFrequency) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L554

```solidity
559: function setUpdateFrequency(bytes32 _key, uint256 _updateFrequency) internal {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L559

```solidity
54: function settleFunds() external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/ValidatorWithdrawalVault.sol#L54



</details>

#### <ins>Recommended Mitigation Steps</ins>

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.



### <a href="#QA Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> block.timestamp is already used when emitting events, no need to input timestamp

#### <ins>Proof Of Concept</ins>

<details>

```solidity
673: emit ERInspectionModeActivated(erInspectionMode, block.timestamp);

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L673



</details>




### <a href="#QA Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Event emit should emit a parameter

Some emitted events do not have any emitted parameters. It is recommended to add some parameter such as state changes or value changes when events are emitted

#### <ins>Proof Of Concept</ins>


<details>


```solidity
500: emit SafeModeEnabled()

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L500

```solidity
505: emit SafeModeDisabled()

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L505


</details>




### <a href="#QA Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Initial value check is missing in Set Functions

A check regarding whether the current value and the new value are the same should be added

#### <ins>Proof Of Concept</ins>

<details>

```solidity
53: function setAdditionalPenaltyAmount(bytes calldata _pubkey, uint256 _amount) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey);
        additionalPenaltyAmount[pubkeyRoot] += _amount;

        emit UpdatedAdditionalPenaltyAmount(_pubkey, _amount);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/Penalty.sol#L53

```solidity
236: function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {
            revert InvalidCommission();
        }
        protocolFee = _protocolFee;
        operatorFee = _operatorFee;

        emit UpdatedCommissionFees(_protocolFee, _operatorFee);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionedPool.sol#L236

```solidity
66: function setCommissionFees(uint256 _protocolFee, uint256 _operatorFee) external {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        if (_protocolFee + _operatorFee > MAX_COMMISSION_LIMIT_BIPS) {
            revert InvalidCommission();
        }
        protocolFee = _protocolFee;
        operatorFee = _operatorFee;

        emit UpdatedCommissionFees(_protocolFee, _operatorFee);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessPool.sol#L66

```solidity
471: function setConstant(bytes32 key, uint256 val) internal {
        constantsMap[key] = val;
        emit SetConstant(key, val);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L471

```solidity
476: function setVariable(bytes32 key, uint256 val) internal {
        variablesMap[key] = val;
        emit SetConstant(key, val);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L476

```solidity
481: function setAccount(bytes32 key, address val) internal {
        UtilLib.checkNonZeroAddress(val);
        accountsMap[key] = val;
        emit SetAccount(key, val);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L481

```solidity
487: function setContract(bytes32 key, address val) internal {
        UtilLib.checkNonZeroAddress(val);
        contractsMap[key] = val;
        emit SetContract(key, val);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L487

```solidity
493: function setToken(bytes32 key, address val) internal {
        UtilLib.checkNonZeroAddress(val);
        tokensMap[key] = val;
        emit SetToken(key, val);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L493

```solidity
515: function setERUpdateFrequency(uint256 _updateFrequency) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        if (_updateFrequency > MAX_ER_UPDATE_FREQUENCY) {
            revert InvalidUpdate();
        }
        setUpdateFrequency(ETHX_ER_UF, _updateFrequency);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L515

```solidity
539: function setSDPriceUpdateFrequency(uint256 _updateFrequency) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        setUpdateFrequency(SD_PRICE_UF, _updateFrequency);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L539

```solidity
544: function setValidatorStatsUpdateFrequency(uint256 _updateFrequency) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        setUpdateFrequency(VALIDATOR_STATS_UF, _updateFrequency);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L544

```solidity
549: function setWithdrawnValidatorsUpdateFrequency(uint256 _updateFrequency) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        setUpdateFrequency(WITHDRAWN_VALIDATORS_UF, _updateFrequency);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L549

```solidity
554: function setMissedAttestationPenaltyUpdateFrequency(uint256 _updateFrequency) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        setUpdateFrequency(MISSED_ATTESTATION_PENALTY_UF, _updateFrequency);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L554

```solidity
559: function setUpdateFrequency(bytes32 _key, uint256 _updateFrequency) internal {
        if (_updateFrequency == 0) {
            revert ZeroFrequency();
        }
        if (_updateFrequency == updateFrequencyMap[_key]) {
            revert FrequencyUnchanged();
        }
        updateFrequencyMap[_key] = _updateFrequency;

        emit UpdateFrequencyUpdated(_updateFrequency);
    }
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L559



</details>





### <a href="#QA Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Missing event for critical parameter change

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
515: function setERUpdateFrequency(uint256 _updateFrequency) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L515

```solidity
539: function setSDPriceUpdateFrequency(uint256 _updateFrequency) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L539

```solidity
544: function setValidatorStatsUpdateFrequency(uint256 _updateFrequency) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L544

```solidity
549: function setWithdrawnValidatorsUpdateFrequency(uint256 _updateFrequency) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L549

```solidity
554: function setMissedAttestationPenaltyUpdateFrequency(uint256 _updateFrequency) external override {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L554



</details>




### <a href="#QA Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Public Functions Not Called By The Contract Should Be Declared External Instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
function getTotalQueuedValidatorCount() public view override returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PermissionlessNodeRegistry.sol#L440

```solidity
function getSocializingPoolAddress(uint8 _poolId)
        public
        view
        override
        onlyExistingPoolId(_poolId)
        returns (address)
    {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PoolUtils.sol#L136

```solidity
function getOperatorTotalNonTerminalKeys(
        uint8 _poolId,
        address _nodeOperator,
        uint256 _startIndex,
        uint256 _endIndex
    ) public view override onlyExistingPoolId(_poolId) returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PoolUtils.sol#L147

```solidity
function convertSDToETH(uint256 _sdAmount) public view override returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/SDCollateral.sol#L205

```solidity
function onlyManagerRole(address account) public view override returns (bool) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L504

```solidity
function onlyOperatorRole(address account) public view override returns (bool) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L508

```solidity
function getERReportableBlock() public view override returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L571

```solidity
function getSDPriceReportableBlock() public view override returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L582

```solidity
function getValidatorStatsReportableBlock() public view override returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L586

```solidity
function getWithdrawnValidatorReportableBlock() public view override returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L590

```solidity
function getExchangeRate() public view override returns (uint256) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderStakePoolsManager.sol#L125

```solidity
function deployWithdrawVault(
        uint8 _poolId,
        uint256 _operatorId,
        uint256 _validatorCount,
        uint256 _validatorId
    ) public override onlyRole(NODE_REGISTRY_CONTRACT) returns (address) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/factory/VaultFactory.sol#L34

```solidity
function deployNodeELRewardVault(uint8 _poolId, uint256 _operatorId)
        public
        override
        onlyRole(NODE_REGISTRY_CONTRACT)
        returns (address)
    {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/factory/VaultFactory.sol#L48

```solidity
function computeWithdrawVaultAddress(
        uint8 _poolId,
        uint256 _operatorId,
        uint256 _validatorCount
    ) public view override returns (address) {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/factory/VaultFactory.sol#L62

```solidity
function computeNodeELRewardVaultAddress(uint8 _poolId, uint256 _operatorId)
        public
        view
        override
        returns (address)
    {
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/factory/VaultFactory.sol#L71



</details>



### <a href="#QA Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Empty blocks should be removed or emit something

#### <ins>Proof Of Concept</ins>

<details>

```solidity
14: constructor() {}

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/NodeELRewardVault.sol#L14

```solidity
23: constructor() {}

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/ValidatorWithdrawalVault.sol#L23

```solidity
17: constructor() {}

```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/VaultProxy.sol#L17



</details>




### <a href="#QA Summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Redundant Cast

The type of the variable is the same as the type to which the variable is being cast

#### <ins>Proof Of Concept</ins>


<details>

```solidity
82: address(_withdrawVault)
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/factory/VaultFactory.sol#L82



</details>





### <a href="#QA Summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Large multiples of ten should use scientific notation

Use (e.g. 1e6) rather than decimal literals (e.g. 100000), for better code readability.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
21: uint256 public constant POOL_WEIGHTS_SUM = 10000;
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/PoolSelector.sol#L21

```solidity
92: setConstant(TOTAL_FEE, 10000);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L92

```solidity
96: setVariable(MAX_DEPOSIT_AMOUNT, 10000 ether);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L96

```solidity
98: setVariable(MAX_WITHDRAW_AMOUNT, 10000 ether);
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderConfig.sol#L98

```solidity
27: uint256 public constant ER_CHANGE_MAX_BPS = 10000;
```

https://github.com/code-423n4/2023-06-stader/tree/main/contracts/StaderOracle.sol#L27



</details>

### <a href="#QA Summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> `calculateRewardShare` will revert when `TOTAL_STAKED_ETH` is 0

When `TOTAL_STAKED_ETH` is 0, the function call will revert due to divison by 0

#### <ins>Proof Of Concept</ins>

This is when assuming the amount of staked ETH per node is currently at 0.

```solidity
uint256 TOTAL_STAKED_ETH = staderConfig.getStakedEthPerNode();
...
uint256 _userShareBeforeCommision = (_totalRewards * usersETH) / TOTAL_STAKED_ETH;
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L255-L261