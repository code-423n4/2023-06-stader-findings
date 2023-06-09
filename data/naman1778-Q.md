## [N-01] Use a modifier for access control

    File: contracts/Auction.sol	

    145: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    152: UtilLib.onlyManagerRole(msg.sender, staderConfig);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol

    File: contracts/ETHx.sol	

    65: UtilLib.onlyManagerRole(msg.sender, staderConfig);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol

    File: contracts/Penalty.sol	

    54: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    63: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    70: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    77: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    84: UtilLib.onlyManagerRole(msg.sender, staderConfig);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol

    File: contracts/PermissionedNodeRegistry.sol	

    145: uint256 operatorId = onlyActiveOperator(msg.sender);

    197: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());

    259: UtilLib.onlyOperatorRole(msg.sender, staderConfig);

    312: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.STADER_ORACLE());

    339: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    352: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    366: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());

    377: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());

    389: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());

    417: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    428: UtilLib.onlyOperatorRole(msg.sender, staderConfig);

    439: UtilLib.onlyOperatorRole(msg.sender, staderConfig);

    450: UtilLib.onlyOperatorRole(msg.sender, staderConfig);

    476: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());

    563: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    572: UtilLib.onlyManagerRole(msg.sender, staderConfig);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol

    File: contracts/PermissionedPool.sol	

    62: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.STADER_INSURANCE_FUND());

    68: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_NODE_REGISTRY());

    90: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.STAKE_POOL_MANAGER());

    130: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_NODE_REGISTRY()); 

    237: UtilLib.onlyManagerRole(msg.sender, staderConfig);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol

    File: contracts/PermissionlessNodeRegistry.sol	

    188: UtilLib.onlyOperatorRole(msg.sender, staderConfig);

    242: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.STADER_ORACLE());

    269: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_POOL());

    280: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_POOL());

    326: UtilLib.onlyOperatorRole(msg.sender, staderConfig);

    337: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    348: UtilLib.onlyOperatorRole(msg.sender, staderConfig);

    382: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_POOL());

    393: UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONLESS_POOL());

    469: UtilLib.onlyManagerRole(msg.sender, staderConfig);

    478: UtilLib.onlyManagerRole(msg.sender, staderConfig);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol

## [N-02] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword

    File: contracts/SocializingPool.sol	

    29: mapping(address => mapping(uint256 => bool)) public override claimedRewards;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol

## [N-03] Use a more recent version of Solidity

For security, it is best practice to use the latest Solidity version.

For the security fix list in the versions: https://github.com/ethereum/solidity/blob/develop/Changelog.md

Context
All contracts

Recommended Mitigation Steps
Old version of Solidity 0.8.16 is used, newer version can be used (0.8.20)

## [N-04] Empty/Unused Function Parameters

Empty or unused function parameters should be commented out as a better and declarative way to silence runtime warning messages. As an example, the following function may have these parameters refactored to:

    File: contracts/StaderStakePoolsManager.sol	

    284: function initialConvertToShares(
    285:     uint256 _assets,
    286:     Math.Rounding /*rounding*/
    287: ) internal pure returns (uint256 shares) {

    305: function initialConvertToAssets(
    306:     uint256 _shares,
    307:     Math.Rounding /*rounding*/
    308: ) internal pure returns (uint256) {

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol

## [N-05] Large multiples of ten should use scientific notation (e.g. 1e6) rather than decimal literals

    File: contracts/PoolSelector.sol	

    21: uint256 public constant POOL_WEIGHTS_SUM = 10000;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol

    File: contracts/StaderConfig.sol	

    92: setConstant(TOTAL_FEE, 10000);

    96: setVariable(MAX_DEPOSIT_AMOUNT, 10000 ether);
    
    98: setVariable(MAX_WITHDRAW_AMOUNT, 10000 ether);

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol    

    File: contracts/StaderOracle.sol	

    27: uint256 public constant ER_CHANGE_MAX_BPS = 10000;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol

    File: contracts/UserWithdrawalManager.sol	
 
    64: maxNonRedeemedUserRequestCount = 1000;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol

