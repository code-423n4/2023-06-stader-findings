instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

contracts\ValidatorWithdrawalVault.sol
31: uint8 poolId = VaultProxy(payable(address(this))).poolId();
32: uint256 validatorId = VaultProxy(payable(address(this))).id(); 
33: IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig(); 

55: uint8 poolId = VaultProxy(payable(address(this))).poolId();
56: uint256 validatorId = VaultProxy(payable(address(this))).id();
57: IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();

contracts\UserWithdrawalManager.sol
104: IERC20Upgradeable(staderConfig.getETHxToken()).safeTransferFrom(msg.sender, (address(this)), _ethXAmount);
recommendation
use hardcoded address.


contracts\PermissionedPool.sol
<x> += <y> Costs more gas Than <X> = <x> + <y> For state Variables
280: preDepositValidatorCount += _count;
285: preDepositValidatorCount -= _count;

