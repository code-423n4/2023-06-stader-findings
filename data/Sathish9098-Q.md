# LOW FINDINGS

## [L-1] Don’t use payable.call()

payable.call() is a low-level method that can be used to send ether to a contract, but it has some limitations and risks as you've pointed out. One of the primary risks of using payable.call() is that it doesn't guarantee that the contract's payable function will be called successfully. This can lead to funds being lost or stuck in the contract

The contract does not have a payable callback
The contract’s payable callback spends more than 2300 gas (which is only enough to emit something)
The contract is called through a proxy which itself uses up the 2300 gas Use OpenZeppelin’s Address.sendValue() instead

```solidity
FILE: 2023-06-stader/contracts/Auction.sol

131:  (bool success, ) = payable(msg.sender).call{value: withdrawalAmount}('');

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#LL131C8-L131C82

```solidity
FILE: 2023-06-stader/contracts/SocializingPool.sol

91:  (bool success, ) = payable(staderConfig.getStaderTreasury()).call{value: _rewardsData.protocolETHRewards} 
     ('');
121: (success, ) = payable(operatorRewardsAddr).call{value: totalAmountETH}('');

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#LL121C13-L121C88

```solidity
FILE: 2023-06-stader/contracts/StaderStakePoolsManager.sol

102: (bool success, ) = payable(staderConfig.getUserWithdrawManager()).call{value: _amount}('');

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderStakePoolsManager.sol#L102

### Recommended Mitigation

Use OpenZeppelin’s Address.sendValue()

##

## [L-2] Use .call instead of .transfer to send ether

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues

````solidity
FILE: 2023-06-stader/contracts/Auction.sol

114:  if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#LL114C8-L114C108

```solidity
FILE: 2023-06-stader/contracts/SDCollateral.sol

68:  if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#LL68C8-L68C96

### Recommended Mitigation

Replace .transfer with .call. Note that the result of .call need to be checked. 


## [L-3] Unbounded loop

### This instance not found by bot race. This is most important Unbounded loop logic 

``nextOperatorId`` the value is only incremented .

PermissionedNodeRegistry.allocateValidatorsAndUpdateOperatorId() will iterate all the OperatorId.

Currently, ``nextOperatorId`` can grow indefinitely. E.g. there’s no maximum limit and there’s no functionality to remove OperatorId

If the value grows too large, calling PermissionedNodeRegistry.allocateValidatorsAndUpdateOperatorId() might run out of gas and revert. 

```solidity
FILE: Breadcrumbs2023-06-stader/contracts/PermissionedNodeRegistry.sol

206:  for (uint256 i = 1; i < nextOperatorId; ) {

488: for (uint256 i = 1; i < nextOperatorId; ) {

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L488

##

## [L-4] Inconsistency between the comment and the actual value assigned

 It appears that there is an inconsistency between the comment and the actual value assigned to the constant variable. The comment states that the value represents a 24-hour duration, but the assigned value of 7200 actually corresponds to a duration of 2 hours

```diff
FILE: Breadcrumbs2023-06-stader/contracts/Auction.sol

- 22: uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours
+ 22: uint256 public constant MIN_AUCTION_DURATION = 7200; // 2 hours
```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#LL22C5-L22C69

##

## [L-5] Resolve the warning to avoid unexpected behavior 

```
Warning (3628): Warning: This contract has a payable fallback function, but no receive ether function. Consider adding a receive ether function.
 --> contracts/VaultProxy.sol:8:1:
  |
8 | contract VaultProxy is IVaultProxy {
  | ^ (Relevant source part starts here and spans across multiple lines).
Note: The payable fallback function is defined here.
  --> contracts/VaultProxy.sol:41:5:
   |
41 |     fallback(bytes calldata _input) external payable returns (bytes memory) {
   |     ^ (Relevant source part starts here and spans across multiple lines).

```
##

## [L-6] Consider using upgradable ERC20 contracts 

Upgradable ERC20 contracts provide flexibility and allow for future upgrades and improvements to the contract's functionality without disrupting the existing ecosystem.

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L9

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L14

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#L15

##

## [L-7] nextValidatorId inrement should be placed on top of the for loop

When increase bottom of the for loop nextValidatorId is increased even next iteration condition check is false.

```solidity
FILE: 2023-06-stader/contracts/PermissionedNodeRegistry.sol

173:  validatorIdByPubkey[_pubkey[i]] = nextValidatorId;
174:            validatorIdsByOperatorId[operatorId].push(nextValidatorId);
175:            emit AddedValidatorKey(msg.sender, _pubkey[i], nextValidatorId);
176:            nextValidatorId++;
177:            unchecked {
178:                ++i;
179:            }

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L173-L179

Recommended Mitigation:
nextValidatorId value should be increased when enter to new iterations 

##

## [L-8] Missing Event for initialize

Description: Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip


```solidity
FILE: 2023-06-stader/contracts/PoolSelector.sol

function initialize(address _admin, address _staderConfig) external initializer {
        UtilLib.checkNonZeroAddress(_admin);
        UtilLib.checkNonZeroAddress(_staderConfig);
        __AccessControl_init_unchained();
        poolAllocationMaxSize = 50;
        staderConfig = IStaderConfig(_staderConfig);
        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
    }

```
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L36-L43

```solidity
FILE: Breadcrumbs2023-06-stader/contracts/VaultProxy.sol

function initialise(
        bool _isValidatorWithdrawalVault,
        uint8 _poolId,
        uint256 _id,
        address _staderConfig
    ) external {
        if (isInitialized) {
            revert AlreadyInitialized();
        }
        UtilLib.checkNonZeroAddress(_staderConfig);
        isValidatorWithdrawalVault = _isValidatorWithdrawalVault;
        isInitialized = true;
        poolId = _poolId;
        id = _id;
        staderConfig = IStaderConfig(_staderConfig);
        owner = staderConfig.getAdmin();
    }


```

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/VaultProxy.sol#LL20C5-L36C6

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/factory/VaultFactory.sol#L23-L32

Recommendation: Add Event-Emit

##


# NON CRITICAL

## [NC-1] Tokens accidentally sent to the contract cannot be recovered

### Context
contracts/staking/NeoTokyoStaker.sol:

It can’t be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps
Add this code:

 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

## [NC-2] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

##

[NC-3] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L23

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L44-L47


##

## [NC-4] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L511-L513

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L581-L583

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L605-L607


##

## [NC-5] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

Consider changing the variable to be an unnamed one

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L76-L79

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L50-L54

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L720

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L513

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L680

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L93

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L432

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L692

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L648

##

## [NC-6] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
(https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

### Recommendation
NatSpec comments should be increased in Contracts


##

## [NC-7] Use a more recent version of solidity

Use at least solidity version 0.8.17 

INSTANCE
ALL CONTRACTS

##

## [NC-8] For functions, follow Solidity standard naming conventions (internal function style rule)

Description
The bellow codes don’t follow Solidity’s standard naming convention,

internal and private functions and variables : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L680

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L687-L692

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L720

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L732

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L738

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L747

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L752

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L758

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L279

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L283

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L288-L293

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L598-L602

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L618

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L625

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L633

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L643-L648

##

## [NC-9] Critical changes should use two-step procedure

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two-step procedure on the critical functions.

Consider adding a two-steps pattern on critical changes to avoid mistakenly transferring ownership of roles or critical functionalities to the wrong address

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L236-L245

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessPool.sol#L66-L75

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L147-L151

##

## [NC-10] add a timelock to critical functions

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L236-L245

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessPool.sol#L66-L75

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L147-L151

##

## [NC-11] No Same value input control

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PoolSelector.sol#L140-L144

##

## [NC-12] Use Fuzzing Test for math code bases

I recommend fuzzing testing in math code structures,

Recommendation:
Use should fuzzing test like Echidna.

Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now.

https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05

##

## [NC-13] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol























