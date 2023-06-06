                                            Finding 1.
 
 There is a gas optimization in https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#59 there is a function createLot here
in line 59. where you increment the value of nextlot with 1 by using nextlot++ . You can Use ++nextlot; to save the gas .

++i costs less gas compared to i++ or i += 1  for an unsigned integer, 
                 as pre-increment is cheaper (about 5 gas per iteration).
This statement is true even with the optimizer enabled.


                            Finding 2 :

Title : ++i costs less gas than i++, especially when it’s used in for-loops .

Saves 5 gas per loop.

There is 1 instance of this issue:

File : code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol 
130:                  i++;

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#130