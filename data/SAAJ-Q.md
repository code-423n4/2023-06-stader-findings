
# Low Risk and Non-Critical Issues

# [L-01] Minting tokens to the zero address should be avoided (01 Instances)

Address(0) check is missing in mint function, consider applying check to ensure tokens aren’t minted to the zero address.

Link to the code:
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L48


# [L-02] Missing event for important parameter change (08 Instances)
Important parameter or configuration changes should trigger an event to allow being tracked off-chain.

Link to the code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L47
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L57
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L140
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L192
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L338
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L351
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L89
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L173

# [L-03] Use a modifier for access control (08 Instances)

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

Link to the code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L64
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L97
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L108
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L207
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L227
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L590
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L625
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L531

# [L-04] Missing initializer modifier on constructor (05 Instances)
OpenZeppelin recommends that the initializer modifier be applied to constructors in order to avoid potential griefs, social engineering, or exploits. Ensure that the modifier is applied to the implementation contract. 
If the default constructor is currently being used, it should be changed to be an explicit one with the modifier applied.

Link to the code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L25
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L25
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L23
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L27
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L62

# [L-05] Add a timelock to critical functions (04 Instances)

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). 

Link to the code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L138
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L144
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L151
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L83



# [L-06] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from various type int/uint values (02 Instances)

Link to the code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L322
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L274


# [L-07] Missing checks for address(0x0) when assigning values to address state variables (12 Instances)

Zero-address check should be used, to avoid the risk of setting a storage variable as address(0) at deploying time.

Link to the code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L140
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L37
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L87
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L57
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L92
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#LL407C69-L407C79
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L47
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L250
9.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L298
10.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessPool.sol#L252
11.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L149
12.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolUtils.sol#L63


# [N-01] According to the syntax rules, use  => mapping ( instead of  => mapping( using spaces as keyword (12 Instances)

Link to the code:
1.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L20
2.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol#L20
3.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L22
4.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L24
5.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L46
6.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L48
7.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L50
8.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L52
9.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L54
10.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L56
11.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L58
12.	https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L59

# [N-02] Pragma Floating (01 Instances)
Locking pragma version ensures contracts are not being deployed on an outdated compiler version.

Link to the code:
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L2


# [N-03] Empty blocks should be removed (01 Instances)

Avoid using code blocks or use them for some process like emitting events.

Link to the code:
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/NodeELRewardVault.sol#L14



# [N-04] Function writing that does not comply with the Solidity Style Guide

All Contracts
Order of functions should help readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered as mentioned in the article i.e.;
constructor
external
public
internal
private
within a grouping, place the view and pure functions last

