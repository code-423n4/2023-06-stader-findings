# Overview
This QA report addresses two key issues within the Stader protocol's smart contract codebase. The first pertains to informational naming and consistency within the Auction contract. The second relates to the ordering of functions and naming of internal functions within the SDCollateral contract. Each of these areas has been identified as requiring attention to align with best practices and improve code readability and maintainability.

# Informational Naming
1) Within the Auction contract, there is an inconsistency in the capitalization of an error message in the [claimSD](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L83) function:
```
if (msg.sender != lotItem.highestBidder) revert notQualified();
```

The lowercase naming of notQualified does not match the usual practice of starting error messages with a capital letter. Therefore, it is recommended that this be updated to NotQualified for consistency and clarity:
```
if (msg.sender != lotItem.highestBidder) revert NotQualified(); 
```


2) The initialise function in the VaultProxy contract should be named initialize to align with the existing function name used in the code. Additionally, the function should have the initializer modifier from the OpenZeppelin library to enforce one-time initialization.
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#L20

# Order of Functions and naming internal function
The order of functions as per their visibility
The placement and naming of the slashSD internal function

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered:
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private

## SDCollateral slashSD internal function
In reviewing the SDCollateral contract, it was noted that the slashSD internal function does not adhere to the recommended order. This function is found within the block of external functions. Specifically, slashSD is situated between two external functions in the codebase, which can potentially cause confusion.

Here is the specific location of the slashSD function in the contract:
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#LL90C14-L90C21

## Internal Function Naming
It is considered a good practice to prefix the names of internal and private functions with an underscore (_). This convention allows developers to immediately recognize the intended visibility of a function, improving clarity and reducing the likelihood of inadvertent visibility errors.


## Recommendations
For maintaining clarity and standard code organization practices, it is recommended that the slashSD function be relocated to the internal functions section of the contract. This change will align the contract with the recommended order and promote better readability and maintainability of the contract code.

By implementing this improvement, the contract would be easier to understand, thus reducing the potential for misunderstanding and bugs. This will also facilitate future code reviews and audits.

function **_slashSD**(address _operator, uint256 _sdToSlash) **internal** {


Here is an example how OpenZeppelin name the internal function:
```
function _transfer(address from, address to, uint256 amount) internal {
```
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/e29e42a529dafcdb5d792cfef97ce71ea0388045/contracts/token/ERC20/ERC20Upgradeable.sol#L222

## Conclusion
Implementing these changes will enhance the clarity and maintainability of the Stader protocol's smart contract code. By aligning with best practices, these improvements will also facilitate future code reviews and audits, ultimately contributing to the robustness and reliability of the protocol.

## Other internal functions without (_) prefix:

**PermissionedNodeRegistry.sol**
- [onboardOperator](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L661)
- [handleFrontRun](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L671)
- [getOperatorQueuedValidatorCount](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L680)
- [checkInputKeysCountAndCollateral](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L687)
- [onlyActiveOperator](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L720)
- [isActiveValidator](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L732)
- [isNonTerminalValidator](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L738)
- [decreaseTotalActiveValidatorCount](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L747)
- [onlyPreDepositValidator](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L656)
- [markValidatorDeposited](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L758)

**PermissionedPool.sol**
- [increasePreDepositValidatorCount](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L279)
- [decreasePreDepositValidatorCount](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L283)
- [preDepositOnBeaconChain](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L288)
- [to_little_endian_64](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L321)

**PermissionlessNodeRegistry.sol**
- onboardOperator
- markKeyReadyToDeposit
- handleFrontRun
- handleInvalidSignature
- checkInputKeysCountAndCollateral
- onlyActiveOperator
- isNonTerminalValidator
- isActiveValidator
- decreaseTotalActiveValidatorCount
- onlyInitializedValidator
- markValidatorDeposited

**PermissionlessPool.sol**
- fullDepositOnBeaconChain
- to_little_endian_64

** PoolUtils.sol**
- verifyNewPool
- getPoolCount

**SDCollateral.sol**
- slashSD
- getOperatorInfo
- isPoolThresholdValid

**StaderConfig.sol**
- setConstant
- setVariable
- setAccount
- setContract
- setToken
- verifyDepositAndWithdrawLimits

**StaderOracle.sol**
- insertSDPrice
- getMedianValue
- setUpdateFrequency
- getReportableBlockFor
- attestSubmission
- getPORFeedData
- updateWithInLimitER
- insertSDPrice

**StaderStakePoolsManager.sol**
- initialConvertToShares

**UserWithdrawalManager.sol**
- deleteRequestId
- sendValue


**ValidatorWithdrawalVault.sol**
- getCollateralETH
- getOperatorAddress

# Order of Layout
Layout contract elements in the following order:
1. Pragma statements
2. Import statements
3. Interfaces
4. Libraries
5. Contracts

Inside each contract, library or interface, use the following order:

1. Type declarations
2. State variables
3. Events
4. Errors
4. Modifiers
5. Functions

## onlyExistingPoolId modifier should be declared before functions
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolUtils.sol#L295

### Description
In the PoolUtils contract, the onlyExistingPoolId modifier is declared after some of the functions. It is recommended to declare modifiers before functions for better readability and consistency.

### Impact
The current order of declaring the onlyExistingPoolId modifier after functions does not have any functional impact on the contract. However, it can make the code harder to read and understand, especially for developers who are not familiar with the codebase.

### Recommendation
To improve code readability and maintainability, it is recommended to declare the onlyExistingPoolId modifier before the functions that use it. This follows a common convention in Solidity contracts and makes it easier for developers to identify and understand the modifiers applied to functions.

```
modifier onlyExistingPoolId(uint8 _poolId) {
    if (!isExistingPoolId(_poolId)) {
        revert PoolIdNotPresent();
    }
    _;
}

function addNewPool(uint8 _poolId, address _poolAddress) external override onlyExistingPoolId(_poolId) {
    // Function implementation
}

function updatePoolAddress(uint8 _poolId, address _newPoolAddress)
    external
    override
    onlyExistingPoolId(_poolId)
    onlyRole(DEFAULT_ADMIN_ROLE)
{
    // Function implementation
}

// Rest of the functions...
```

By declaring the onlyExistingPoolId modifier before the functions that use it, the code follows a more standard and readable structure.


### List of Modifier after functions

**StaderOracle.sol**
- checkERInspectionMode
- trustedNodeOnly
- checkMinTrustedNodes

**VaultProxy.sol**
- onlyOwner

# Underscore Prefix for Non-external Functions and Variables
The contract does not consistently use an underscore prefix for private functions and variables, as recommended by the Solidity [style guide](https://docs.soliditylang.org/en/v0.8.20/style-guide.html). It is good practice to use an underscore prefix to clearly distinguish private functions and variables from public and external ones. Consider adding an underscore prefix to private functions and variables throughout the contract.

## List of private Variables without _ prefix
**StaderConfig.sol**
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderConfig.sol#L74-L78
- mapping(bytes32 => uint256) private constantsMap;
- mapping(bytes32 => uint256) private variablesMap;
- mapping(bytes32 => address) private accountsMap;
- mapping(bytes32 => address) private contractsMap;
- mapping(bytes32 => address) private tokensMap;

**PoolUtils.sol**
- uint64 private constant PUBKEY_LENGTH = 48;
- uint64 private constant SIGNATURE_LENGTH = 96;

**StaderOracle**
- mapping(bytes32 => bool) private nodeSubmissionKeys;
- mapping(bytes32 => uint8) private submissionCountKeys;
- uint256[] private sdPrices;

