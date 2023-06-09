[G-01] Use != 0 instead of > 0 for Unsigned Integer Comparison

When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.

There are 13 instances of this issue:

File: contracts/Auction.sol

    109: if (lotItem.highestBidAmount > 0) revert LotWasAuctioned();

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L109

File: contracts/PermissionedNodeRegistry.sol

    204: bool validatorPerOperatorGreaterThanZero = (validatorPerOperator > 0);

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L204

    299: if (totalDefectedKeys > 0) {

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L299

File: contracts/PermissionlessNodeRegistry.sol

    209: if (frontRunValidatorsLength > 0) {

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L209

    305: if (address(feeRecipientAddress).balance > 0) {

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionlessNodeRegistry.sol#L305

File: contracts/SocializingPool.sol

    119: if (totalAmountETH > 0) {

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#L119

    127: if (totalAmountSD > 0) {

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SocializingPool.sol#L127

File: contracts/StaderOracle.sol

    121: if (_exchangeRate.reportingBlockNumber % updateFrequencyMap[ETHX_ER_UF] > 0) {

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L121

    274: if (_sdPriceData.reportingBlockNumber % updateFrequencyMap[SD_PRICE_UF] > 0) {

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L274

    328: if (_validatorStats.reportingBlockNumber % updateFrequencyMap[VALIDATOR_STATS_UF] > 0) {

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L328

    403: if (_withdrawnValidators.reportingBlockNumber % updateFrequencyMap[WITHDRAWN_VALIDATORS_UF] > 0) {

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L403

File: contracts/StaderStakePoolsManager.sol

    330: (totalAssets() > 0 ||

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderStakePoolsManager.sol#L330

File: contracts/ValidatorWithdrawalVault.sol

    115: if (totalRewards > 0) {

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/ValidatorWithdrawalVault.sol#L115


[G-02]  <x> += <y> and <x> -= <y> cost more gas than <x> = <x> + <y> and <x> = <x> - <y> for state variables.

There are 5 unknown instances of this issue:

File: contracts/OperatorRewardsCollector.sol

	41: balances[_receiver] += msg.value;

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/OperatorRewardsCollector.sol#L41

	49: balances[operator] -= amount;

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/OperatorRewardsCollector.sol#L49

File: contracts/Penalty.sol

	56: additionalPenaltyAmount[pubkeyRoot] += _amount;

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Penalty.sol#L56

File: contracts/SDCollateral.sol

	45: operatorSDBalance[operator] += _sdAmount;

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L45

	96: operatorSDBalance[_operator] -= sdSlashed;

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L96