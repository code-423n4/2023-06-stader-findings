 
# Gas Optimizations Report

This report focuses on Stader Protocol contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of stader protocol are categorised into 10 main areas; with further multiple instances in each of the category.

# [G-01] Immutable has more gas efficiency than constant (15 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost.

Link to the Code:

1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L22
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L31
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L33
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L30
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L42
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L43
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L23
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L31
9.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L21
10.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L13
11.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L14
12.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L26
13.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L27
14.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L29
15.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L18


# [G-02] Use hardcode address instead address(this) (27 Instances)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

Link to the code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L55
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L25
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L26
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L27
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L28
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L174
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L178
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L47
9.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L69
10.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L75
11.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L43
12.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol#L62
13.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L189
14.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L221
15.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L104
16.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L155
17.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L224
18.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L31
19.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L32
20.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L33
21.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L34
22.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L55
23.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L56
24.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L57
25.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L94
26.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L95
27.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L99



# [G-03] Use != 0 instead of > 0 for unsigned integer comparison (13 Instances)
	
Link to the Code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L109
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L204
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L299
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L209
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L305
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L119
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L127
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L121
9.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L274
10.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L328
11.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L403
12.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L330
13.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L115


# [G-04] Use uint256(1)/uint256(2) instead for true and false boolean states (14 Instances)

Boolean for storage if not used, saves Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

Link to the Code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L86
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L101
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L94
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L80
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L155
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L87
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L100
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L191
9.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L499
10.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L504
11.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L628
12.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L668
13.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L75
14.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L31


# [G-05] Setting the constructor to payable (11 Instances)

Saves ~13 gas per instance

Link to the Code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L25
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L25
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L14
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L23
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L27
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L62
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L36
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L62
9.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L34
10.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L26
11.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L21


# [G-06] Public functions to external instead (12 Instances)

Functions with public visibility, if not called within the contract needed to be changed external.

Link to the Code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#LL440C14-L440C42
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L504
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderConfig.sol#L508
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L571
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L582
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L586
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L590
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L164
9.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L34
10.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L48
11.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L62
12.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L71


# [G 07] abi.encode() is less efficient than abi.encodepacked()(18 Instances)

Refer to this article.

Link to the Code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L127
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L135
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L221
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L233
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L282
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L283
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L334
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L346
9.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L407
10.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L410
11.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L418
12.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L466
13.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L469
14.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L470
15.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L40
16.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L54
17.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L67
18.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L77


# [G 08] String literals passed to abi.encode()/abi.encodePacked() should not be split by commas (26 Instances)

String literals can be split into multiple parts and still be considered as a single string literal.
EACH new comma costs 21 gas due to stack operations and separate MSTOREs.
	
Link to the Code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L263
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L267
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L273
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L274
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L229
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L235
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L236
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L174
9.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L127
10.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L135
11.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L221
12.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L233
13.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L282
14.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L283
15.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L334
16.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L346
17.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L410
18.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L418
19.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L469
20.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L470
21.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L40
22.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L54
23.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L67
24.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L77
25.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L82
26.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L140


# [G-09] Use assembly to check for address(0) (02 Instances)
	
Link to the Code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/UserWithdrawalManager.sol#L96
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol#L22



# [G-10] Use calldata instead of memory for function parameters (01 Instances)

Using calldata in external function does not require data to be stored, which reduced the process time as compared to memory. This in return saves gas during calling the data.

Link to the Code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SDCollateral.sol#L124
