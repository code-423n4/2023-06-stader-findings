## [G-01] abi.encode() is less efficient than abi.encodePacked()

    File: contracts/StaderOracle.sol	

    127: abi.encode(

    135: abi.encode(_exchangeRate.reportingBlockNumber, _exchangeRate.totalETHBalance, _exchangeRate.totalETHXSupply)

    221: abi.encode(

    233: abi.encode(

    282: bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _sdPriceData.reportingBlockNumber));

    283: bytes32 submissionCountKey = keccak256(abi.encode(_sdPriceData.reportingBlockNumber));

    334: abi.encode(

    346: abi.encode(

    407: bytes memory encodedPubkeys = abi.encode(_withdrawnValidators.sortedPubkeys);

    410: abi.encode(

    418: abi.encode(_withdrawnValidators.reportingBlockNumber, _withdrawnValidators.nodeRegistry, encodedPubkeys)

    466: bytes memory encodedPubkeys = abi.encode(_mapd.sortedPubkeys);

    469: bytes32 nodeSubmissionKey = keccak256(abi.encode(msg.sender, _mapd.index, encodedPubkeys));

    470: bytes32 submissionCountKey = keccak256(abi.encode(_mapd.index, encodedPubkeys));

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

    File: contracts/factory/VaultFactory.sol	

    40: bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));

    54: bytes32 salt = sha256(abi.encode(_poolId, _operatorId));

    67: bytes32 salt = sha256(abi.encode(_poolId, _operatorId, _validatorCount));

    77: bytes32 salt = sha256(abi.encode(_poolId, _operatorId));

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

    | Contract0 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 101871                                    | 683             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |
    
    
    | Contract1 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 99465                                     | 671             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-02] Use assembly to write address storage values

    File: contracts/Penalty.sol	

    43: ratedOracleAddress = _ratedOracleAddress;

    86: ratedOracleAddress = _ratedOracleAddress;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol

    File: contracts/VaultProxy.sol	

    35: owner = staderConfig.getAdmin();
 
    70: owner = _owner;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol

    File: contracts/factory/VaultFactory.sol	

    29: vaultProxyImplementation = address(new VaultProxy());

    95: vaultProxyImplementation = _vaultProxyImpl;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.setAddress(0x0000000000000000000000000000000000000001);
            c1.setAddressOptimized(0x0000000000000000000000000000000000000001);
        }
    }
    contract Contract0 {
        address a;
        function setAddress(address _a) public returns(bytes32){
            a = _a;
        }
    }
    contract Contract1 {
        address a;
        function setAddressOptimized(address _a) public returns(bytes32){
            assembly
            {
                sstore(a.slot, _a)
            }
        }
    }

#### Gas Report

    | src/test/GasTest.t.sol:Contract0 contract |                 |       |        |       |         |
    |-------------------------------------------|-----------------|-------|--------|-------|---------|
    | Deployment Cost                           | Deployment Size |       |        |       |         |
    | 51905                                     | 291             |       |        |       |         |
    | Function Name                             | min             | avg   | median | max   | # calls |
    | setAddress                                | 22411           | 22411 | 22411  | 22411 | 1       |
    
    
    | src/test/GasTest.t.sol:Contract1 contract |                 |       |        |       |         |
    |-------------------------------------------|-----------------|-------|--------|-------|---------|
    | Deployment Cost                           | Deployment Size |       |        |       |         |
    | 39093                                     | 226             |       |        |       |         |
    | Function Name                             | min             | avg   | median | max   | # calls |
    | setAddressOptimized                       | 22378           | 22378 | 22378  | 22378 | 1       |

## [G-03] Use assembly to check for address(0)
   
    File: contracts/UserWithdrawalManager.sol	

    96: if (_owner == address(0)) revert ZeroAddressReceived();

if (_owner == address(0)) revert ZeroAddressReceived();

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(0x0000000000000000000000000000000000000001);
            c1.optimized(0x0000000000000000000000000000000000000001);
        }
    }
    
    contract Contract0 {
    
         function not_optimized(address addr) public pure{
             if(addr == address(0)){
                revert();
             }
         }
    }
    
    contract Contract1 {
    
         function optimized(address addr) public pure{
             assembly {
               if iszero(addr) {
                   revert(0x00, 0x20)
               }
            }
         }
    }

#### Gas Report

    | src/test/GasTest.t.sol:Contract0 contract |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 41893                                     | 240             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | not_optimized                             | 258             | 258 | 258    | 258 | 1       |
    
    
    | src/test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 37687                                     | 219             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | optimized                                 | 252             | 252 | 252    | 252 | 1       |

## [G-04] Use nested if and, avoid multiple check combinations

Using nesting is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

    File: contracts/SocializingPool.sol	

    146: if (_amountSD[i] == 0 && _amountETH[i] == 0) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol

    File: contracts/StaderConfig.sol	

    514: if (
    515:     !(variablesMap[MIN_DEPOSIT_AMOUNT] != 0 &&
    516:         variablesMap[MIN_WITHDRAW_AMOUNT] != 0 &&
    517:         variablesMap[MIN_DEPOSIT_AMOUNT] <= variablesMap[MAX_DEPOSIT_AMOUNT] &&
    518:         variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MAX_WITHDRAW_AMOUNT] &&
    519:         variablesMap[MIN_WITHDRAW_AMOUNT] <= variablesMap[MIN_DEPOSIT_AMOUNT] &&
    520:         variablesMap[MAX_WITHDRAW_AMOUNT] >= variablesMap[MAX_DEPOSIT_AMOUNT])
    521: ) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol

    File: contracts/StaderOracle.sol	

    147: if (
    148:     submissionCount == trustedNodesCount / 2 + 1 &&
    149:     _exchangeRate.reportingBlockNumber > exchangeRate.reportingBlockNumber
    150: ) {

    186: if (
    187:     !staderConfig.onlyManagerRole(msg.sender) &&
    188:     erInspectionModeStartBlock + MAX_ER_UPDATE_FREQUENCY > block.number
    189: ) {

    371: if (
    372:     submissionCount == trustedNodesCount / 2 + 1 &&
    373:     _validatorStats.reportingBlockNumber > validatorStats.reportingBlockNumber
    374: ) {

    431: if (
    432:     submissionCount == trustedNodesCount / 2 + 1 &&
    433:     _withdrawnValidators.reportingBlockNumber > reportingBlockNumberForWithdrawnValidators
    434: ) {

    664: if (
    665:     !(newExchangeRate >= (currentExchangeRate * (ER_CHANGE_MAX_BPS - erChangeLimit)) / ER_CHANGE_MAX_BPS &&
    666:        newExchangeRate <= ((currentExchangeRate * (ER_CHANGE_MAX_BPS + erChangeLimit)) / ER_CHANGE_MAX_BPS))
    667: ) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

    File: contracts/ValidatorWithdrawalVault.sol	

    35: if (!staderConfig.onlyOperatorRole(msg.sender) && totalRewards > staderConfig.getRewardsThreshold()) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }

    contract Contract0 {

        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }

    }

    contract Contract1 {

        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Report

    | Contract0 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 76923                                     | 416             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | checkAge                                  | 651             | 651 | 651    | 651 | 1       |


    | Contract1 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 76323                                     | 413             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |

## [G-05] Splitting *require()* statements that use && saves gas

Instead of using single rquire statment with multiple and operators checking the condition make use of multiple require statements

    File: contracts/StaderStakePoolsManager.sol	

    331: IStaderOracle(staderConfig.getStaderOracle()).getExchangeRate().totalETHXSupply == 0) && (!paused());

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }

    contract Contract0 {

        function checkAge(uint8 _age) public returns(string memory){
            require(_age>18 && _age<22);
            return "Eligible";
        }

    }

    contract Contract1 {

        function checkAgeOptimized(uint8 _age) public returns(string memory){
            require(_age>18);
            require(_age<22);
            return "Eligible";
        }
    }

#### Gas Report 

    | Contract0 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 76723                                     | 415             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | checkAge                                  | 649             | 649 | 649    | 649 | 1       |


    | Contract1 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 76923                                     | 416             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | checkAgeOptimized                         | 641             | 641 | 641    | 641 | 1       |

## [G-06] Instead of counting down in *for* statements, count up // check for loop not starting with 0

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

    File: contracts/Penalty.sol	

    100: for (uint256 i; i < reportedValidatorCount; ) {
    117: ++i;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol

    File: contracts/PermissionedNodeRegistry.sol	

    91: for (uint256 i = 0; i < permissionedNosLength; i++) {

    155: for (uint256 i; i < keyCount; ) {
    178: ++i; 

    206: for (uint256 i = 1; i < nextOperatorId; ) {
    215: ++i;

    272: for (uint256 i; i < frontRunValidatorsLength; ) {
    280: ++i;

    285: for (uint256 i; i < invalidSignatureValidatorsLength; ) {
    293: ++i;

    317: for (uint256 i; i < withdrawnValidatorCount; ) {
    327: ++i;

    488: for (uint256 i = 1; i < nextOperatorId; ) {
    493: ++i;

    534: for (uint256 i = _startIndex; i < _endIndex; ) {
    540: ++i;

    598: for (uint256 i = startIndex; i < endIndex; i++) {

    637: for (uint256 i = startIndex; i < endIndex; i++) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

    File: contracts/PermissionedPool.sol	

    137: for (uint256 i; i < pubkeyCount; ) {
    164: ++i;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol

    File: contracts/PermissionlessNodeRegistry.sol	

    140: for (uint256 i; i < keyCount; ) {
    164: ++i;

    199: for (uint256 i; i < readyToDepositValidatorsLength; ) {
    205: ++i;

    215: for (uint256 i; i < frontRunValidatorsLength; ) {
    221: ++i;

    225: for (uint256 i; i < invalidSignatureValidatorsLength; ) {
    231: ++i;

    247: for (uint256 i; i < withdrawnValidatorCount; ) {
    257: ++i;

    415: for (uint256 i = _startIndex; i < _endIndex; ) {
    421: ++i;

    504: for (uint256 i = startIndex; i < endIndex; i++) {

    543: for (uint256 i = startIndex; i < endIndex; i++) {

    573: for (uint256 i = startIndex; i < endIndex; i++) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

    File: contracts/PermissionlessPool.sol	

    94: for (uint256 i; i < pubkeyCount; ) {
    117: ++i;

    137: for (uint256 i = depositQueueStartIndex; i < requiredValidators + depositQueueStartIndex; ) {
    147: ++i;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol

    File: contracts/PoolSelector.sol	

    90: for (uint256 j; j < poolCount; ) {
    108: ++j;

    130: for (uint256 i = 0; i < poolTargetLength; i++) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol

    File: contracts/PoolUtils.sol	

    75: for (uint256 i; i < exitValidatorCount; ) {
    
    104: for (uint256 i = 0; i < poolCount; i++) {

    168: for (uint256 i = 0; i < poolCount; i++) {

    179: for (uint256 i = 0; i < poolCount; i++) {
    
    190: for (uint256 i = 0; i < poolCount; i++) {

    201: for (uint256 i = 0; i < poolCount; i++) {

    273: for (uint256 i = 0; i < poolCount; i++) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol

    File: contracts/SocializingPool.sol	

    145: for (uint256 i = 0; i < indexLength; i++) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol
    
    File: contracts/StaderOracle.sol	

    485: for (uint256 i; i < keyCount; ) {
    489: ++i;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

    File: contracts/StaderStakePoolsManager.sol	

    232: for (uint256 i = 0; i < poolCount; i++) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol
 
    File: contracts/UserWithdrawalManager.sol	
 
    132: for (requestId = nextRequestIdToFinalize; requestId <= maxRequestIdToFinalize; ) {
    149: ++requestId;

    210: for (uint256 i; i < userRequestCount; ) {
    217: ++i;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }

    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }

    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

    | Contract0 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 77011                                     | 311             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


    | Contract1 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 73811                                     | 295             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-07] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

    File: contracts/SDCollateral.sol	

    119: function updatePoolThreshold(
    121:     uint8 _poolId,
    122:     uint256 _minThreshold,
    123:     uint256 _maxThreshold,
    124:     uint256 _withdrawThreshold,
    125:     string memory _units
    126: ) external override {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }
    
    contract Contract0 {
    
         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }
    
    contract Contract1 {
    
         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

    | Contract0 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 100747                                    | 535             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | not_optimized                             | 790             | 790 | 790    | 790 | 1       |
    
    
    | Contract1 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 66917                                     | 366             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | optimized                                 | 556             | 556 | 556    | 556 | 1       |


## [G-08] Using XOR (^) and OR (|) bitwise equivalents instead of != and || operators

    File: contracts/PermissionedNodeRegistry.sol	

    693: if (_pubkeyLength != _preDepositSignatureLength || _pubkeyLength != _depositSignatureLength) {

    734: return (validator.status == ValidatorStatus.PRE_DEPOSIT || validator.status == ValidatorStatus.DEPOSITED);

    740: return
    741:    !(validator.status == ValidatorStatus.WITHDRAWN ||
    742:        validator.status == ValidatorStatus.FRONT_RUN ||
    743:        validator.status == ValidatorStatus.INVALID_SIGNATURE);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

    File: contracts/PermissionlessNodeRegistry.sol	

    649: if (_pubkeyLength != _preDepositSignatureLength || _pubkeyLength != _depositSignatureLength) {

    693: return
    694:     !(validator.status == ValidatorStatus.WITHDRAWN ||
    695:        validator.status == ValidatorStatus.FRONT_RUN ||
    696:        validator.status == ValidatorStatus.INVALID_SIGNATURE);  //// @audit use bitwise operators

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

    File: contracts/StaderStakePoolsManager.sol	

    274: (_assets == 0 || supply == 0)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol
 
    File: contracts/library/UtilLib.sol	

    161: uint256 newExchangeRate = (totalETHBalance == 0 || totalETHXSupply == 0)

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(5,10,16);
            c1.optimized(5,10,16);
        }
    }
    contract Contract0 {
        address a;
        function not_optimized(uint8 x,uint8 y, uint8 z) public returns(bool){
            if(x!=5 || y!=10 || z!=15){
                return true;
            }
        }
    }
    contract Contract1 {
        address a;
        function optimized(uint8 x,uint8 y, uint8 z) public returns(bool){
            if(((x^5) | (y^10) | (z^15)) != 0){
                return true;
            }
        }
    }
    
#### Gas Report

    | src/test/GasTest.t.sol:Contract0 contract |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 53705                                     | 300             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | not_optimized                             | 622             | 622 | 622    | 622 | 1       |
    
    
    | src/test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 49899                                     | 280             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | optimized                                 | 569             | 569 | 569    | 569 | 1       |

## [G-09] Avoid using state variables in emit // Not yet completed

    File: contracts/Auction.sol	

    44: emit AuctionDurationUpdated(duration);

    45: emit BidIncrementUpdated(bidIncrement);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol

    File: contracts/PermissionedNodeRegistry.sol	

    419: emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);

    430: emit UpdatedInputKeyCountLimit(inputKeyCountLimit);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

    File: contracts/PermissionlessNodeRegistry.sol	

    271: emit UpdatedNextQueuedValidatorIndex(nextQueuedValidatorIndex);

    328: emit UpdatedInputKeyCountLimit(inputKeyCountLimit);

    339: emit UpdatedMaxNonTerminalKeyPerOperator(maxNonTerminalKeyPerOperator);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

## [G-10] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

    File: contracts/Penalty.sol	

    106: uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);

    107: uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol