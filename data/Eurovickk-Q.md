https://github.com/code-423n4/2023-06-stader/blob/main/contracts/library/UtilLib.sol

The getPubkeyForValidSender, getOperatorForValidSender, and onlyValidatorWithdrawVault functions check if _addr matches withdrawVaultAddress, but they revert with CallerNotWithdrawVault if they don't match. This could be misleading since the error message suggests that the caller is not a withdraw vault, but it could also mean that the caller is not the expected address. 

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol

In the poolAllocationForExcessETHDeposit function, the for loop uses both i and j variables, but j is not initialized or used within the loop. It seems like j should be used instead of i for the loop iteration.

In the poolAllocationForExcessETHDeposit function, the unchecked keyword is used to suppress the integer overflow check when incrementing j. Be cautious when using unchecked and ensure that the logic is correct to avoid unintended bugs related to overflow.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderInsuranceFund.sol

n the withdrawFund function, the contract balance is checked against _amount before transferring funds. However, the comparison should use the greater-than-or-equal-to operator (>=) instead of just the greater-than operator (>). This ensures that the contract has enough funds to cover the withdrawal amount.

In the withdrawFund function, the _amount parameter is compared against zero (_amount == 0) to check if it's a valid amount. It's generally recommended to use a require statement with a meaningful error message instead of a revert statement when validating input parameters. For example: require(_amount > 0, "Invalid amount provided");

In the withdrawFund and reimburseUserFund functions, the contract checks the balance using address(this).balance before performing the transfer. It's important to consider reentrancy issues when handling funds. The ReentrancyGuardUpgradeable from OpenZeppelin is being used, which is a good practice.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol

In the distributeRewards function, the condition !staderConfig.onlyOperatorRole(msg.sender) is used to check if the sender is not an operator. However, the correct syntax should be !staderConfig.isOperatorRole(msg.sender). Make sure the isOperatorRole function returns true if the sender has the operator role.

In the distributeRewards function, the revert InvalidRewardAmount() statement is missing a new keyword before InvalidRewardAmount. It should be revert new InvalidRewardAmount(); to create a new instance of the exception.

In the settleFunds function, the condition msg.sender != nodeRegistry is used to check if the caller is not the node registry contract. However, there is no definition provided for the CallerNotNodeRegistryContract exception. You should define and emit an appropriate exception in this case.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol

In the fallback function, when the delegatecall to the vault implementation fails, the error message is converted from bytes to string using string(data). However, this conversion may not always be successful, and it's generally safer to revert with the original data bytes. Consider using revert(data) instead.

-----------------------------------------------------------------------------------------------------------------

   function initialize(address _admin, address _staderConfig) external initializer {
        UtilLib.checkNonZeroAddress(_admin);
        UtilLib.checkNonZeroAddress(_staderConfig);
        __AccessControl_init_unchained();

        staderConfig = IStaderConfig(_staderConfig);
        vaultProxyImplementation = address(new VaultProxy());

        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
    }

The initialize function uses the _admin parameter to grant the DEFAULT_ADMIN_ROLE, but it doesn't check if the caller is the contract creator. Consider adding a check to ensure that only the contract creator can call this function.