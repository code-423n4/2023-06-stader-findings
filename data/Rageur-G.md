## GAS-1: 10 ** X can be changed to 1eX

### Affected file

* StaderConfig.sol (Line: 93, 95, 97)

## GAS-2: Add unchecked{} for subtractions where the operands cannot underflow because of a previous require() or if statement

### Description

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
 if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
 This will stop the check for overflow and underflow so it will save gas.

### Affected file

* PermissionedNodeRegistry.sol (Line: 224)

## GAS-3: Caching global variables is more expensive than using the actual variable

### Description

 It’s cheaper to use global variable as compared to caching.

### Affected file

* OperatorRewardsCollector.sol (Line: 47)
* SDCollateral.sol (Line: 44, 59)
* SocializingPool.sol (Line: 48, 113)
* StaderOracle.sol (Line: 669)
* StaderStakePoolsManager.sol (Line: 56, 170, 241)

## GAS-4: Change ```public``` state variable visibility to ```private```

### Description

If it is preferred to change the visibility of the ```owner``` state variable to private, this will save significant gas.

### Affected file

* VaultProxy.sol (Line: 14)

## GAS-5: Do not calculate constants

### Description

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

### Affected file

* ETHx.sol (Line: 21, 22)
* StaderConfig.sol (Line: 12, 14, 16, 18, 20, 22, 24, 25, 27, 28, 29, 30, 31, 33, 35, 37, 38, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 60, 64, 65, 68, 69, 71, 72)
* StaderOracle.sol (Line: 26, 50, 51, 52, 53, 54)
* VaultFactory.sol (Line: 16)

## GAS-6: Duplicated require()/revert() checks should be refactored to a modifier or function

### Description

This saves deployment gas.

### Affected file

* Auction.sol (Line: 82, 97, 108, 122)
* NodeELRewardVault.sol (Line: 29)
* PermissionedNodeRegistry.sol (Line: 125, 207, 227, 264, 314, 340, 527, 536, 590, 599, 625, 631, 693, 693, 697, 697, 702, 708, 722, 725)
* PermissionedPool.sol (Line: 102, 238)
* PermissionlessNodeRegistry.sol (Line: 102, 192, 244, 408, 417, 496, 505, 531, 537, 565, 649, 649, 653, 653, 659, 668, 682, 685)
* PermissionlessPool.sol (Line: 68)
* PoolUtils.sol (Line: 170, 181, 192, 203)
* SocializingPool.sol (Line: 92, 122, 170, 207)
* StaderInsuranceFund.sol (Line: 43, 49, 62)
* StaderOracle.sol (Line: 147, 255, 371, 431, 482)
* StaderStakePoolsManager.sol (Line: 103, 234)
* UserWithdrawalManager.sol (Line: 224, 230)
* UtilLib.sol (Line: 59, 75, 90, 169)
* ValidatorWithdrawalVault.sol (Line: 39)
* VaultProxy.sol (Line: 46)

## GAS-7: Empty blocks should be removed or emit something

### Description

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

### Affected file

* NodeELRewardVault.sol (Line: 14)
* ValidatorWithdrawalVault.sol (Line: 23)
* VaultProxy.sol (Line: 17)

## GAS-8: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* ETHx.sol (Line: 29)
* PermissionedNodeRegistry.sol (Line: 66)
* PermissionedPool.sol (Line: 40)
* PermissionlessNodeRegistry.sol (Line: 66, 440)
* PermissionlessPool.sol (Line: 38)
* PoolUtils.sol (Line: 25)
* SDCollateral.sol (Line: 26, 205)
* SocializingPool.sol (Line: 39)
* StaderConfig.sol (Line: 504, 508)
* StaderInsuranceFund.sol (Line: 26)
* StaderOracle.sol (Line: 62, 571, 582, 586, 590)
* StaderStakePoolsManager.sol (Line: 50, 164)
* UserWithdrawalManager.sol (Line: 54)
* VaultFactory.sol (Line: 34, 48, 62, 71)

## GAS-9: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* PoolUtils.sol (Line: 295)
* StaderOracle.sol (Line: 697, 704, 711)
* VaultProxy.sol (Line: 75)

## GAS-10: Should use arguments instead of state variable

### Description

Using function's parameter cost less gas then a state variable.

### Affected file

* Auction.sol (Line: 148)
* PermissionedNodeRegistry.sol (Line: 419, 430)
* PermissionlessNodeRegistry.sol (Line: 271, 328, 339)
* StaderOracle.sol (Line: 536)
* VaultFactory.sol (Line: 96)
* VaultProxy.sol (Line: 71)

## GAS-11: Use ```assembly``` to write address storage values

### Affected file

* Auction.sol (Line: 37, 38, 39, 40, 53, 66, 81, 94, 107, 121, 140, 147, 153)
* ETHx.sol (Line: 37, 87)
* OperatorRewardsCollector.sol (Line: 34, 58)
* Penalty.sol (Line: 42, 43, 44, 45, 46, 64, 71, 78, 86, 93)
* PermissionedNodeRegistry.sol (Line: 72, 73, 74, 75, 76, 77, 78, 79, 238, 418, 429, 440, 451, 461)
* PermissionedPool.sol (Line: 45, 46, 47, 241, 242, 250)
* PermissionlessNodeRegistry.sol (Line: 72, 73, 74, 75, 76, 77, 270, 327, 338, 349, 356)
* PermissionlessPool.sol (Line: 43, 44, 45, 71, 72, 212)
* PoolSelector.sol (Line: 40, 41, 104, 142, 149)
* PoolUtils.sol (Line: 29, 86)
* SDCollateral.sol (Line: 33, 81, 115, 147, 173, 197)
* SocializingPool.sol (Line: 47, 48, 84, 181)
* StaderInsuranceFund.sol (Line: 31, 71)
* StaderOracle.sol (Line: 69, 75, 192, 291, 375, 435, 483, 499, 504, 511, 525, 535, 668, 669)
* StaderStakePoolsManager.sol (Line: 56, 57, 58, 111, 118, 241)
* UserWithdrawalManager.sol (Line: 60, 61, 62, 63, 64, 79, 86, 154, 209)
* ValidatorWithdrawalVault.sol (Line: 75)
* VaultFactory.sol (Line: 28, 29, 88, 95)
* VaultProxy.sol (Line: 30, 31, 32, 33, 34, 35, 59, 70)

## GAS-12: Use constants instead of type(uintx).max

### Description

It uses more gas in the distribution process and also for each transaction than constant usage.

### Affected file

* SDCollateral.sol (Line: 106)

## GAS-13: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* Auction.sol (Line: 55)
* NodeELRewardVault.sol (Line: 25, 26, 27, 28)
* PermissionedPool.sol (Line: 174, 178)
* SDCollateral.sol (Line: 47)
* SocializingPool.sol (Line: 69, 75)
* StaderInsuranceFund.sol (Line: 43, 62)
* StaderStakePoolsManager.sol (Line: 189, 221)
* UserWithdrawalManager.sol (Line: 104, 155, 224)
* ValidatorWithdrawalVault.sol (Line: 31, 32, 33, 34, 55, 56, 57, 94, 95, 99)

## GAS-14: Use nested if and avoid multiple check combinations

### Description

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

### Affected file

* SocializingPool.sol (Line: 146)
* StaderConfig.sol (Line: 513)
* StaderOracle.sol (Line: 147, 186, 371, 431, 664)
* ValidatorWithdrawalVault.sol (Line: 35)

## GAS-15: Use selfbalance()

### Description

Use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.

### Affected file

* NodeELRewardVault.sol (Line: 28)
* PermissionedPool.sol (Line: 174, 178)
* SocializingPool.sol (Line: 69)
* StaderInsuranceFund.sol (Line: 43, 62)
* StaderStakePoolsManager.sol (Line: 189, 221)
* UserWithdrawalManager.sol (Line: 224)
* ValidatorWithdrawalVault.sol (Line: 34, 99)

## GAS-16: Using > 0 costs more gas than != 0 when used on a uint in a require() statement

### Description

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance.

### Affected file

* Auction.sol (Line: 109)
* PermissionedNodeRegistry.sol (Line: 299)
* PermissionlessNodeRegistry.sol (Line: 209, 305)
* SocializingPool.sol (Line: 119, 127)
* StaderOracle.sol (Line: 121, 274, 328, 403)
* ValidatorWithdrawalVault.sol (Line: 115)

## GAS-17: With assembly, ```.call (bool success)``` transfer can be done gas-optimized

### Description

```return``` data ```(bool success,)``` has to be stored due to EVM architecture, but in a usage in assembly, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

### Affected file

* Auction.sol (Line: 131)
* SocializingPool.sol (Line: 91, 121)
* StaderInsuranceFund.sol (Line: 48)
* StaderStakePoolsManager.sol (Line: 102)
* UserWithdrawalManager.sol (Line: 229)
* UtilLib.sol (Line: 168)
* VaultProxy.sol (Line: 45)

## GAS-18: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* StaderOracle.sol (Line: 127, 135, 221, 233, 282, 283, 334, 346, 407, 410, 418, 466, 469, 470)
* VaultFactory.sol (Line: 40, 54, 67, 77)