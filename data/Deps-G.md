# Access control upgradeable oversized

Problem:
`import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol'` control access is using unnecessary gas for the roles existing in the `contracts/factory/VaultFactory.sol` contract. 
The whole access control is used only for `DEFAULT_ADMIN_ROLE` and `NODE_REGISTRY_CONTRACT`.

Solution:
Replace `AccessControlUpgradeable.sol` contract for `mapping (address => bool) public DEFAULT_ADMIN_ROLE` and `mapping (address => bool) public NODE_REGISTRY_CONTRACT`, this way the gas cost to check the user Role will be inferior and the contract size also, saving gas in the transaction and crypto in the deployment.