## Summary

### Low Risk Issues List

| Number | Issues Details                                                                                                              | Context |
| :----: | :-------------------------------------------------------------------------------------------------------------------------- | :-----: |
| [L-01] | `_safeMint()` should be used rather than `_mint()` wherever possible                                                        |    1    |
| [L-02] | Missing Event for `initialize`                                                                                              |   12    |
| [L-03] | It is safer to change the `owner address` with the `Ownable2Step.sol` pattern                                               |    1    |
| [L-04] | `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` |    5    |
| [L-05] | Contracts are not using their `OZ Upgradeable` counterparts                                                                 |    1    |
| [L-06] | Upgrade OpenZeppelin` Contract Dependency                                                                                   |    1    |
| [L-07] | No input validation `ratedOracleAddress` address on Penalty.updateRatedOracleAddress function                               |    1    |
| [L-08] | no check if the burn amount is zero or not                                                                                  |    1    |

## [L-01] `_safeMint()` should be used rather than `_mint()` wherever possible

`_mint()` is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. Both [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) and [solmate](https://github.com/Rari-Capital/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180) have versions of this function

_There is one instance of this issue:_

```solidity
File: contracts/ETHx.sol
47    function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) whenNotPaused {
48        _mint(to, amount);
49    }

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L48

## [L-02] Missing Event for `initialize`

Description: Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip

Recommendation: Add Event-Emit

```solidity
File:  contracts/PermissionedNodeRegistry.sol

66    function initialize(address _admin, address _staderConfig) public initializer {
67        UtilLib.checkNonZeroAddress(_admin);
68        UtilLib.checkNonZeroAddress(_staderConfig);
69        __AccessControl_init_unchained();
70        __Pausable_init();
71        __ReentrancyGuard_init();
72        staderConfig = IStaderConfig(_staderConfig);
73        nextOperatorId = 1;
74        nextValidatorId = 1;
75        operatorIdForExcessDeposit = 1;
76        inputKeyCountLimit = 50;
77        maxOperatorId = 10;
78        maxNonTerminalKeyPerOperator = 50;
79        verifiedKeyBatchSize = 50;
80        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
81    }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L66

```solidity
File:   contracts/PermissionedPool.sol

40    function initialize(address _admin, address _staderConfig) public initializer {
41        UtilLib.checkNonZeroAddress(_admin);
42        UtilLib.checkNonZeroAddress(_staderConfig);
43        __AccessControl_init_unchained();
44        __ReentrancyGuard_init();
45        protocolFee = 500;
46        operatorFee = 500;
47        staderConfig = IStaderConfig(_staderConfig);
48        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
49    }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L40

```solidity
File:  contracts/PermissionlessNodeRegistry.sol


66    function initialize(address _admin, address _staderConfig) public initializer {
67        UtilLib.checkNonZeroAddress(_admin);
68        UtilLib.checkNonZeroAddress(_staderConfig);
69        __AccessControl_init_unchained();
70        __Pausable_init();
71        __ReentrancyGuard_init();
72        staderConfig = IStaderConfig(_staderConfig);
73        nextOperatorId = 1;
74        nextValidatorId = 1;
75        inputKeyCountLimit = 30;
76        maxNonTerminalKeyPerOperator = 50;
77        verifiedKeyBatchSize = 50;
78        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
79    }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L66

```solidity
File: contracts/PermissionlessPool.sol


38    function initialize(address _admin, address _staderConfig) public initializer {
39        UtilLib.checkNonZeroAddress(_admin);
40        UtilLib.checkNonZeroAddress(_staderConfig);
41        __AccessControl_init_unchained();
42        __ReentrancyGuard_init();
43        protocolFee = 500;
44        operatorFee = 500;
45       staderConfig = IStaderConfig(_staderConfig);
46        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
47    }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L38

```solidity
File:  contracts/PoolSelector.sol

36    function initialize(address _admin, address _staderConfig) external initializer {
37        UtilLib.checkNonZeroAddress(_admin);
38        UtilLib.checkNonZeroAddress(_staderConfig);
39        __AccessControl_init_unchained();
40        poolAllocationMaxSize = 50;
41        staderConfig = IStaderConfig(_staderConfig);
42        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
43    }

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L36

```solidity
File:  contracts/PoolUtils.sol


25    function initialize(address _admin, address _staderConfig) public initializer {
26        UtilLib.checkNonZeroAddress(_admin);
27        UtilLib.checkNonZeroAddress(_staderConfig);
28        __AccessControl_init_unchained();
29        staderConfig = IStaderConfig(_staderConfig);
30
31        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
32    }

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L25

```solidity
File:  contracts/SocializingPool.sol


39    function initialize(address _admin, address _staderConfig) public initializer {
40        UtilLib.checkNonZeroAddress(_admin);
41        UtilLib.checkNonZeroAddress(_staderConfig);
42
43        __AccessControl_init();
44        __Pausable_init();
45        __ReentrancyGuard_init();
46
47        staderConfig = IStaderConfig(_staderConfig);
48        initialBlock = block.number;
49
50        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
51    }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L39

```solidity
File:  contracts/StaderConfig.sol


85    function initialize(address _admin, address _ethDepositContract) external initializer {
86        UtilLib.checkNonZeroAddress(_admin);
87       UtilLib.checkNonZeroAddress(_ethDepositContract);
88        __AccessControl_init();
89        setConstant(ETH_PER_NODE, 32 ether);
90        setConstant(PRE_DEPOSIT_SIZE, 1 ether);
91        setConstant(FULL_DEPOSIT_SIZE, 31 ether);
92        setConstant(TOTAL_FEE, 10000);
93        setConstant(DECIMALS, 10**18);
94        setConstant(OPERATOR_MAX_NAME_LENGTH, 255);
95        setVariable(MIN_DEPOSIT_AMOUNT, 10**14);
96        setVariable(MAX_DEPOSIT_AMOUNT, 10000 ether);
97        setVariable(MIN_WITHDRAW_AMOUNT, 10**14);
98        setVariable(MAX_WITHDRAW_AMOUNT, 10000 ether);
99        setVariable(WITHDRAWN_KEYS_BATCH_SIZE, 50);
100        setVariable(MIN_BLOCK_DELAY_TO_FINALIZE_WITHDRAW_REQUEST, 600);
101        setContract(ETH_DEPOSIT_CONTRACT, _ethDepositContract);
102       _grantRole(DEFAULT_ADMIN_ROLE, _admin);
103   }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L85

```solidity
File: contracts/StaderInsuranceFund.sol


26    function initialize(address _admin, address _staderConfig) public initializer {
27        UtilLib.checkNonZeroAddress(_admin);
28        UtilLib.checkNonZeroAddress(_staderConfig);
29        __AccessControl_init_unchained();
30        __ReentrancyGuard_init();
31        staderConfig = IStaderConfig(_staderConfig);
32        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
33    }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L26

```solidity
File:  contracts/StaderStakePoolsManager.sol


50    function initialize(address _admin, address _staderConfig) public initializer {
51        UtilLib.checkNonZeroAddress(_admin);
52        UtilLib.checkNonZeroAddress(_staderConfig);
53        __AccessControl_init();
54        __Pausable_init();
55        __ReentrancyGuard_init();
56       lastExcessETHDepositBlock = block.number;
57        excessETHDepositCoolDown = 3 * 7200;
58        staderConfig = IStaderConfig(_staderConfig);
59        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
60    }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L50

```solidity
File: contracts/UserWithdrawalManager.sol


54    function initialize(address _admin, address _staderConfig) public initializer {
55        UtilLib.checkNonZeroAddress(_admin);
56        UtilLib.checkNonZeroAddress(_staderConfig);
57        __AccessControl_init_unchained();
58       __Pausable_init();
59        __ReentrancyGuard_init();
60        staderConfig = IStaderConfig(_staderConfig);
61        nextRequestIdToFinalize = 1;
62        nextRequestId = 1;
63        finalizationBatchLimit = 50;
64        maxNonRedeemedUserRequestCount = 1000;
65       _grantRole(DEFAULT_ADMIN_ROLE, _admin);
66    }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L54

```solidity
File: contracts/factory/VaultFactory.sol


23    function initialize(address _admin, address _staderConfig) external initializer {
24        UtilLib.checkNonZeroAddress(_admin);
25        UtilLib.checkNonZeroAddress(_staderConfig);
26        __AccessControl_init_unchained();
27
28        staderConfig = IStaderConfig(_staderConfig);
29        vaultProxyImplementation = address(new VaultProxy());
30
31        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
32    }

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L23

## [L-03] It is safer to change the `owner address` with the `Ownable2Step.sol` pattern

The owner address change is done with the updateOwner function, but there is a risk of incorrect address definition here, it is a serious risk for the platform, so it is safer to do it with the Ownable2Step pattern.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L68

```solidity
File:  contracts/VaultProxy.sol


68    function updateOwner(address _owner) external override onlyOwner {
69        UtilLib.checkNonZeroAddress(_owner);
70        owner = _owner;
71        emit UpdatedOwner(owner);
72    }
```

## [L‑04] `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`

Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). "Unless there is a compelling reason, abi.encode should be preferred". If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.

If all arguments are strings and or bytes, bytes.concat() should be used instead

There are 5 instances of this issue:

```solidity
File: contracts/PermissionedPool.sol

/// @audit _pubkey
263:         bytes32 pubkey_root = sha256(abi.encodePacked(_pubkey, bytes16(0)));


/// @audit _pubkey
255:         bytes32 pubkey_root = sha256(abi.encodePacked(_pubkey, bytes16(0)));

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol

```solidity
File: contracts/SocializingPool.sol

/// @audit _operator, _amountSD, _amountETH
174        bytes32 node = keccak256(abi.encodePacked(_operator, _amountSD, _amountETH));

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L174

```solidity
File: contracts/factory/VaultFactory.sol

/// @audit   _withdrawVault
82        return abi.encodePacked(bytes1(0x01), bytes11(0x0), address(_withdrawVault));

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L82

```solidity
File: contracts/library/UtilLib.sol


/// @audit   _pubkey
140         return sha256(abi.encodePacked(_pubkey, bytes16(0)));

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L140

## [L-05] Contracts are not using their `OZ Upgradeable` counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts. It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

```solidity
File:

9:   import '@openzeppelin/contracts/token/ERC20/IERC20.sol'
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L9

```solidity
File:

14:   import '@openzeppelin/contracts/token/ERC20/IERC20.sol';

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L14

```solidity
File:

15:     import '@openzeppelin/contracts/token/ERC20/IERC20.sol';
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L15

## [L-06] Upgrade OpenZeppelin` Contract Dependency

```js

37:    "@openzeppelin/contracts": "^4.7.3",

```

```js
VULNERABILITY	VULNERABLE VERSION
M
Incorrect Calculation
>=4.8.0 <4.8.2
M
Incorrect Calculation
>=4.8.0 <4.8.2
H
Improper Verification of Cryptographic Signature
<4.7.3
M
Denial of Service (DoS)
>=3.2.0 <4.7.2
L
Incorrect Resource Transfer Between Spheres
>=4.6.0 <4.7.2
H
Incorrect Calculation
>=4.3.0 <4.7.2
H
Information Exposure
>=4.1.0 <4.7.1
H
Information Exposure
>=4.0.0 <4.7.1
```

## [L-07] No input validation `ratedOracleAddress` address on Penalty.updateRatedOracleAddress function

every one can set an address

```solidity
File: contracts/Penalty.sol

83    function updateRatedOracleAddress(address _ratedOracleAddress) external override {
84        UtilLib.onlyManagerRole(msg.sender, staderConfig);
85        UtilLib.checkNonZeroAddress(_ratedOracleAddress);
86        ratedOracleAddress = _ratedOracleAddress;
87        emit UpdatedPenaltyOracleAddress(_ratedOracleAddress);
88    }
89
90    //update the address of staderConfig
90    function updateStaderConfig(address _staderConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {
90        UtilLib.checkNonZeroAddress(_staderConfig);
90        staderConfig = IStaderConfig(_staderConfig);
90        emit UpdatedStaderConfig(_staderConfig);
90    }
```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#LL83C1-L95C6

## [L-08] no check if the burn amount is zero or not

if the amount is zero so the unhecked block will divided zero on 3 and we use gas for nothing ! if we set zero we may pass the `_burn checks`, i know it is passed by only permit but it's better to avoid this happen because it's seting by human and it means it can be set with 0 balance to burn !

```solidity

56    function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {
57        _burn(account, amount);
58    }

```

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L56
