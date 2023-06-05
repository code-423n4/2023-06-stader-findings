
# VULN 1 

## [LOW] Keccak Constant values should used to immutable rather than constant
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 21 at 2023-06-stader-labs/contracts/ETHx.sol:

    bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');


Found in line 22 at 2023-06-stader-labs/contracts/ETHx.sol:

    bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');


Found in line 12 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant ETH_PER_NODE = keccak256('ETH_PER_NODE');


Found in line 14 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant PRE_DEPOSIT_SIZE = keccak256('PRE_DEPOSIT_SIZE');


Found in line 16 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant FULL_DEPOSIT_SIZE = keccak256('FULL_DEPOSIT_SIZE');


Found in line 18 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant DECIMALS = keccak256('DECIMALS');


Found in line 20 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant TOTAL_FEE = keccak256('TOTAL_FEE');


Found in line 22 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant OPERATOR_MAX_NAME_LENGTH = keccak256('OPERATOR_MAX_NAME_LENGTH');


Found in line 24 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant SOCIALIZING_POOL_CYCLE_DURATION = keccak256('SOCIALIZING_POOL_CYCLE_DURATION');


Found in line 27 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant REWARD_THRESHOLD = keccak256('REWARD_THRESHOLD');


Found in line 28 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant MIN_DEPOSIT_AMOUNT = keccak256('MIN_DEPOSIT_AMOUNT');


Found in line 29 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant MAX_DEPOSIT_AMOUNT = keccak256('MAX_DEPOSIT_AMOUNT');


Found in line 30 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant MIN_WITHDRAW_AMOUNT = keccak256('MIN_WITHDRAW_AMOUNT');


Found in line 31 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant MAX_WITHDRAW_AMOUNT = keccak256('MAX_WITHDRAW_AMOUNT');


Found in line 35 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant WITHDRAWN_KEYS_BATCH_SIZE = keccak256('WITHDRAWN_KEYS_BATCH_SIZE');


Found in line 37 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant ADMIN = keccak256('ADMIN');


Found in line 38 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant STADER_TREASURY = keccak256('STADER_TREASURY');


Found in line 40 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');


Found in line 41 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');


Found in line 42 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');


Found in line 43 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');


Found in line 44 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');


Found in line 45 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');


Found in line 46 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');


Found in line 47 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');


Found in line 48 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');


Found in line 49 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');


Found in line 50 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override ETH_DEPOSIT_CONTRACT = keccak256('ETH_DEPOSIT_CONTRACT');


Found in line 51 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONLESS_POOL = keccak256('PERMISSIONLESS_POOL');


Found in line 52 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');


Found in line 53 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');


Found in line 54 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');


Found in line 55 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');


Found in line 56 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');


Found in line 57 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');


Found in line 64 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');


Found in line 65 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');


Found in line 68 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override MANAGER = keccak256('MANAGER');


Found in line 69 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override OPERATOR = keccak256('OPERATOR');


Found in line 71 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant SD = keccak256('SD');


Found in line 72 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant ETHx = keccak256('ETHx');


Found in line 50 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency


Found in line 51 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key


Found in line 52 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key


Found in line 53 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key


Found in line 54 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key


Found in line 16 at 2023-06-stader-labs/contracts/factory/VaultFactory.sol:

    bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');

------------------------------------------------------------------------ 

### Mitigation 

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.










# VULN 2 

## [LOW] Empty receive()/payable fallback() function does not authenticate requests
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 52 at 2023-06-stader-labs/contracts/PermissionedPool.sol:

    receive() external payable {


Found in line 26 at 2023-06-stader-labs/contracts/ValidatorWithdrawalVault.sol:

    receive() external payable {


Found in line 68 at 2023-06-stader-labs/contracts/StaderStakePoolsManager.sol:

    receive() external payable {


Found in line 57 at 2023-06-stader-labs/contracts/SocializingPool.sol:

    receive() external payable {


Found in line 50 at 2023-06-stader-labs/contracts/PermissionlessPool.sol:

    receive() external payable {


Found in line 20 at 2023-06-stader-labs/contracts/NodeELRewardVault.sol:

    receive() external payable {


Found in line 68 at 2023-06-stader-labs/contracts/UserWithdrawalManager.sol:

    receive() external payable {

------------------------------------------------------------------------ 

### Mitigation 

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))).Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas. Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.










# VULN 3 

## [LOW] Keccak Constant values should used to immutable rather than constant
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 21 at 2023-06-stader-labs/contracts/ETHx.sol:

    bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');


Found in line 22 at 2023-06-stader-labs/contracts/ETHx.sol:

    bytes32 public constant BURNER_ROLE = keccak256('BURNER_ROLE');


Found in line 12 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant ETH_PER_NODE = keccak256('ETH_PER_NODE');


Found in line 14 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant PRE_DEPOSIT_SIZE = keccak256('PRE_DEPOSIT_SIZE');


Found in line 16 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant FULL_DEPOSIT_SIZE = keccak256('FULL_DEPOSIT_SIZE');


Found in line 18 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant DECIMALS = keccak256('DECIMALS');


Found in line 20 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant TOTAL_FEE = keccak256('TOTAL_FEE');


Found in line 22 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant OPERATOR_MAX_NAME_LENGTH = keccak256('OPERATOR_MAX_NAME_LENGTH');


Found in line 24 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant SOCIALIZING_POOL_CYCLE_DURATION = keccak256('SOCIALIZING_POOL_CYCLE_DURATION');


Found in line 27 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant REWARD_THRESHOLD = keccak256('REWARD_THRESHOLD');


Found in line 28 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant MIN_DEPOSIT_AMOUNT = keccak256('MIN_DEPOSIT_AMOUNT');


Found in line 29 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant MAX_DEPOSIT_AMOUNT = keccak256('MAX_DEPOSIT_AMOUNT');


Found in line 30 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant MIN_WITHDRAW_AMOUNT = keccak256('MIN_WITHDRAW_AMOUNT');


Found in line 31 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant MAX_WITHDRAW_AMOUNT = keccak256('MAX_WITHDRAW_AMOUNT');


Found in line 35 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant WITHDRAWN_KEYS_BATCH_SIZE = keccak256('WITHDRAWN_KEYS_BATCH_SIZE');


Found in line 37 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant ADMIN = keccak256('ADMIN');


Found in line 38 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant STADER_TREASURY = keccak256('STADER_TREASURY');


Found in line 40 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override POOL_UTILS = keccak256('POOL_UTILS');


Found in line 41 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override POOL_SELECTOR = keccak256('POOL_SELECTOR');


Found in line 42 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override SD_COLLATERAL = keccak256('SD_COLLATERAL');


Found in line 43 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override OPERATOR_REWARD_COLLECTOR = keccak256('OPERATOR_REWARD_COLLECTOR');


Found in line 44 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override VAULT_FACTORY = keccak256('VAULT_FACTORY');


Found in line 45 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override STADER_ORACLE = keccak256('STADER_ORACLE');


Found in line 46 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override AUCTION_CONTRACT = keccak256('AuctionContract');


Found in line 47 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PENALTY_CONTRACT = keccak256('PENALTY_CONTRACT');


Found in line 48 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONED_POOL = keccak256('PERMISSIONED_POOL');


Found in line 49 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override STAKE_POOL_MANAGER = keccak256('STAKE_POOL_MANAGER');


Found in line 50 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override ETH_DEPOSIT_CONTRACT = keccak256('ETH_DEPOSIT_CONTRACT');


Found in line 51 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONLESS_POOL = keccak256('PERMISSIONLESS_POOL');


Found in line 52 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override USER_WITHDRAW_MANAGER = keccak256('USER_WITHDRAW_MANAGER');


Found in line 53 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override STADER_INSURANCE_FUND = keccak256('STADER_INSURANCE_FUND');


Found in line 54 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONED_NODE_REGISTRY = keccak256('PERMISSIONED_NODE_REGISTRY');


Found in line 55 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONLESS_NODE_REGISTRY = keccak256('PERMISSIONLESS_NODE_REGISTRY');


Found in line 56 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONED_SOCIALIZING_POOL = keccak256('PERMISSIONED_SOCIALIZING_POOL');


Found in line 57 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override PERMISSIONLESS_SOCIALIZING_POOL = keccak256('PERMISSIONLESS_SOCIALIZING_POOL');


Found in line 64 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override ETH_BALANCE_POR_FEED = keccak256('ETH_BALANCE_POR_FEED');


Found in line 65 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override ETHX_SUPPLY_POR_FEED = keccak256('ETHX_SUPPLY_POR_FEED');


Found in line 68 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override MANAGER = keccak256('MANAGER');


Found in line 69 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant override OPERATOR = keccak256('OPERATOR');


Found in line 71 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant SD = keccak256('SD');


Found in line 72 at 2023-06-stader-labs/contracts/StaderConfig.sol:

    bytes32 public constant ETHx = keccak256('ETHx');


Found in line 50 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant ETHX_ER_UF = keccak256('ETHX_ER_UF'); // ETHx Exchange Rate, Balances Update Frequency


Found in line 51 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant SD_PRICE_UF = keccak256('SD_PRICE_UF'); // SD Price Update Frequency Key


Found in line 52 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant VALIDATOR_STATS_UF = keccak256('VALIDATOR_STATS_UF'); // Validator Status Update Frequency Key


Found in line 53 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant WITHDRAWN_VALIDATORS_UF = keccak256('WITHDRAWN_VALIDATORS_UF'); // Withdrawn Validator Update Frequency Key


Found in line 54 at 2023-06-stader-labs/contracts/StaderOracle.sol:

    bytes32 public constant MISSED_ATTESTATION_PENALTY_UF = keccak256('MISSED_ATTESTATION_PENALTY_UF'); // Missed Attestation Penalty Update Frequency Key


Found in line 16 at 2023-06-stader-labs/contracts/factory/VaultFactory.sol:

    bytes32 public constant override NODE_REGISTRY_CONTRACT = keccak256('NODE_REGISTRY_CONTRACT');

------------------------------------------------------------------------ 

### Mitigation 

By using immutable variables, the Keccak256 hash is computed at contract deployment time rather than at compilation time. This means that the value can be updated if the algorithm changes in a future compiler version, without breaking backward compatibility. Additionally, it provides better gas optimization, as the Keccak256 hash is computed only once at contract deployment instead of every time the variable is accessed during execution.










# VULN 4 

## [LOW] Use .call instead of .transfer to send ether
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 87 at 2023-06-stader-labs/contracts/Auction.sol:

        if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {


Found in line 114 at 2023-06-stader-labs/contracts/Auction.sol:

        if (!IERC20(staderConfig.getStaderToken()).transfer(staderConfig.getStaderTreasury(), _sdAmount)) {


Found in line 129 at 2023-06-stader-labs/contracts/SocializingPool.sol:

            if (!IERC20(staderConfig.getStaderToken()).transfer(operatorRewardsAddr, totalAmountSD)) {


Found in line 68 at 2023-06-stader-labs/contracts/SDCollateral.sol:

        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {

------------------------------------------------------------------------ 

### Mitigation 

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues.Replace .transfer with .call. Note that the result of .call need to be checked.










# VULN 5 

## [LOW] Use the safe variant and ERC721.mint
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 48 at 2023-06-stader-labs/contracts/ETHx.sol:

        _mint(to, amount);

------------------------------------------------------------------------ 

### Mitigation 

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset. OpenZeppelin recommendation is to use the safe variant of _mint. Replace _mint() with _safeMint().
