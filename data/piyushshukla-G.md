Increase gas costs on all onlyRole operations

Due to how constant variables are implemented, a keccak256 expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas (about 100 per access).

If the variable was immutable instead: the calculation would only be done once at deploy time (in the constructor), and then the result would be saved and read directly at runtime rather than being recalculated.

See: ethereum/solidity#9232

 bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');
 bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');

code = https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ETHx.sol#LL21-L22


### Recommended
Change the variable to be immutable rather than constant