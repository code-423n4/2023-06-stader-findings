## Bid Withdrawal Mechanism in Auction Contract
The current design of the auction contract includes the [`withdrawUnselectedBid`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L120-L135) function, which allows withdrawal of funds only after the auction ends. While this safeguards the auction process from disruptions, it locks up participants' funds for a prolonged period.

To enhance the fluidity of the auction, it is proposed to refactor the [`addBid`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#LL62C1-L78C6) function, thereby eliminating the need for `withdrawUnselectedBid`. In the refactored `addBid`, the previous highest bidder's bid will be deleted from `lotItem.bids[lotItem.highestBidder]`, and the bidder's funds will be refunded before updating `lotItem.highestBidder`, `lotItem.highestBidAmount`, and `lotItem.bids[msg.sender]`.

The refactored addBid function could look like this:

```solidity
function addBid(uint256 lotId) external payable override whenNotPaused {
    // reject payments of 0 ETH
    if (msg.value == 0) revert InSufficientETH();

    LotItem storage lotItem = lots[lotId];
    if (block.number > lotItem.endBlock) revert AuctionEnded();

    uint256 totalUserBid = lotItem.bids[msg.sender] + msg.value;

    if (totalUserBid < lotItem.highestBidAmount + bidIncrement) revert InSufficientBid();

    // Refund previous highest bidder and delete their bid before updating
    if(lotItem.highestBidder != address(0)) {
        payable(lotItem.highestBidder).transfer(lotItem.highestBidAmount);
        delete lotItem.bids[lotItem.highestBidder];
    }

    lotItem.highestBidder = msg.sender;
    lotItem.highestBidAmount = totalUserBid;
    lotItem.bids[msg.sender] = totalUserBid;

    emit BidPlaced(lotId, msg.sender, totalUserBid);
}
```
This refactor makes sure that before a new highest bid is accepted, the previous highest bid is refunded and removed from lotItem.bids. It's important to note that this design would need a proper audit and testing to make sure it behaves as expected in all scenarios. Additionally, take into account that the refund operation could fail due to gas constraints, so a more robust mechanism to handle such scenarios might be needed.

## Streamlining the addBid function in Auction Contract: Removal of Redundant Code
The current implementation of the `addBid` function in the Auction contract contains a line of code that checks whether the sent ETH amount (`msg.value`) is zero. Specifically, the line [`if (msg.value == 0) revert InSufficientETH();`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L64) appears to be redundant due to the subsequent condition that checks if the total bid is less than the highest current bid plus the bid increment.

In the event `msg.value` is zero, the latter condition [`if (totalUserBid < lotItem.highestBidAmount + bidIncrement) revert InSufficientBid();`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol#L71) would effectively handle the scenario, as it encompasses the case where no new ETH is added to the bid.

While separate error messages could aid in user experience by precisely pointing out the mistake, in this context, it seems superfluous as a user not sending any ETH with the bid and a user not meeting the minimum bid increment inherently suggest the same issue - the bid is insufficient.

Therefore, it is recommended to remove the line `if (msg.value == 0) revert InSufficientETH();` from the `addBid` function to simplify the code and remove redundancy, without compromising the function's integrity or user experience.

## Initiating Role Management in VaultFactory Contract Initialization and Subsequent Function Call Permissions
The VaultFactory contract could greatly benefit from incorporating a role management function for the NODE_REGISTRY_CONTRACT role within the `initialize` function. This initial role assignment is key to ensuring secure and authorized contract functionality from the onset.

Upon contract deployment, it's crucial that functions [`deployWithdrawVault`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L39) and [`deployNodeELRewardVault`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L51) are callable, particularly considering their `onlyRole(NODE_REGISTRY_CONTRACT) visibility`. Setting the NODE_REGISTRY_CONTRACT role within the `initialize` function can facilitate this.

Here's a sample implementation within the initialize function:

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/factory/VaultFactory.sol#L23-L32

```diff
function initialize(address _admin, address _staderConfig) external initializer {
    ...
    _grantRole(DEFAULT_ADMIN_ROLE, _msgSender());
+    _grantRole(NODE_REGISTRY_CONTRACT, _msgSender());
}
```
The same shall apply to the ['MINTER_ROLE' and 'BURNER_ROLE'](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ETHx.sol#L21-L22) in the ETHx contract too.

## Strict Reporting Block Number Updates and Consideration for Potential Delays
The StaderOracle contract enforces strict reporting block number updates based on predetermined update frequencies. For example, the update frequencies are set as follows:

[. ETHX_ER_UF: 7200 blocks
. SD_PRICE_UF: 7200 blocks
. VALIDATOR_STATS_UF: 7200 blocks
. WITHDRAWN_VALIDATORS_UF: 14400 blocks](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L70-L73)

To ensure accurate data, the reporting block numbers must be updated strictly at block numbers that are multiples of their respective update frequencies. However, it's important to consider potential delays that may prevent timely updates.

For instance, let's take the ETHX_ER_UF data type as an example. If the current reporting block number is not a multiple of 7200, it indicates a missed update. The next update for this data type would need to occur at the next multiple of 7200 blocks, resulting in a potential delay in data availability. Otherwise, calling function `submitExchangeRateData` is going to readily revert:

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L121-L123

```solidity
        if (_exchangeRate.reportingBlockNumber % updateFrequencyMap[ETHX_ER_UF] > 0) {
            revert InvalidReportingBlock();
        }
```
Contract operators should be aware of potential delays caused by blockchain congestion or other factors. They should closely monitor the reporting block numbers and strive to meet the update frequencies to avoid data becoming stale. If frequent delays are encountered, it may be necessary to adjust the chosen update frequencies to better align with the blockchain's performance and ensure timely and accurate data updates.

## Comments and Code Mismatch
The withdraw function in the SDCollateral contract lacks the implementation of a withdrawal delay, as indicated in the function's NatSpec documentation. This omission contradicts the intended behavior specified in the documentation, potentially leading to inconsistencies and unintended consequences in the withdrawal process.

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#L54-L73

```solidity
    /// @notice for operator to request withdraw of sd
    /// @dev it does not transfer sd tokens immediately
    /// operator should come back after withdrawal-delay time to claim
    /// this requested sd is subject to slashes
    function withdraw(uint256 _requestedSD) external override {
        address operator = msg.sender;
        uint256 opSDBalance = operatorSDBalance[operator];

        if (opSDBalance < getOperatorWithdrawThreshold(operator) + _requestedSD) {
            revert InsufficientSDToWithdraw(opSDBalance);
        }
        operatorSDBalance[operator] -= _requestedSD;

        // cannot use safeERC20 as this contract is an upgradeable contract
        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
            revert SDTransferFailed();
        }

        emit SDWithdrawn(operator, _requestedSD);
    }
```
It is recommended to address this discrepancy by implementing the appropriate withdrawal delay mechanism in the withdraw function. By adhering to the intended withdrawal delay, operators can claim their requested SD tokens only after the specified time period, ensuring the proper functioning of the collateral slashing mechanism and maintaining the expected behavior of the contract.

## Identical Code Logic
In ValidatorWithdrawalVault.sol, functions `distributeRewards` and `settleFunds` have the following identical code snippet:

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L31-L33
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L55-L57

```solidity
        uint8 poolId = VaultProxy(payable(address(this))).poolId();
        uint256 validatorId = VaultProxy(payable(address(this))).id();
        IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
```
It is recommended to refactor the duplicate code snippet by grouping it into a private function. This will improve code maintainability and reduce redundancy. By extracting the common code snippet into a separate function, you can avoid duplicating the same code and make future updates or modifications easier.

## Missing Length Check in claim Function
The [`claim`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L107-L135) function in the SocializingPool contract should include a length check for the input parameters `_index`, `_amountSD`, `_amountETH`, and `_merkleProof` before invoking the [`_claim`](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/SocializingPool.sol#L137-L161) internal function. This check ensures that all arrays have the same length, preventing potential errors or inconsistencies during the reward claiming process.

By verifying the lengths of these arrays, you can ensure that the corresponding values align properly and avoid any unexpected behavior or unintended consequences. Adding this length check will help maintain the integrity of the claim operation and enhance the overall robustness of the contract.

Consider adding the following length check at the beginning of the claim function:

```solidity
require(
    _index.length == _amountSD.length &&
    _index.length == _amountETH.length &&
    _index.length == _merkleProof.length,
    "Input lengths mismatch"
);
```
## Unnecessary payable Keyword in withdraw Function
The `payable` keyword in the line `if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD))` of the `withdraw` function in the `SDCollateral` contract is unnecessary and can be safely removed.

Since you're not dealing with the transfer of native tokens (ETH) in this specific line, using the `payable` keyword is not required. The `payable` keyword is typically used when transferring ETH or performing operations related to ETH transfers.

You can simplify the line to:

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#LL68C1-L68C96

```diff
-        if (!IERC20(staderConfig.getStaderToken()).transfer(payable(operator), _requestedSD)) {
+        if (!IERC20(staderConfig.getStaderToken()).transfer(operator, _requestedSD)) {
```
By removing the `payable` keyword, you ensure clarity and eliminate any confusion regarding the transfer of native tokens, making the code more concise and accurate.

## Removal of Unused receive and fallback Functions
The `receive` and `fallback` functions in the contract `PermissionedPool` and `StaderStakePoolsManager.sol` can be safely removed as they serve as protection against accidental submissions by calling non-existent functions. By removing these functions, direct transfers of Ether to the contract will be disallowed, achieving the same outcome as having the functions present with the reverting logic.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L51-L59
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderStakePoolsManager.sol#L62-L70

```solidity
    // protection against accidental submissions by calling non-existent function
    receive() external payable {
        revert UnsupportedOperation();
    }

    // protection against accidental submissions by calling non-existent function
    fallback() external payable {
        revert UnsupportedOperation();
    }
```
## Stale Values Attributable to Unpredictably Delayed Updates
The following concensus in StaderOracle.sol will have to be exactly met in order to have the update logics executed, highly leading to stale answers:

1. function `submitExchangeRateData`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L147-L156

```solidity
        if (
            submissionCount == trustedNodesCount / 2 + 1 &&
            _exchangeRate.reportingBlockNumber > exchangeRate.reportingBlockNumber
        ) {
            updateWithInLimitER(
                _exchangeRate.totalETHBalance,
                _exchangeRate.totalETHXSupply,
                _exchangeRate.reportingBlockNumber
            );
        }
``` 
2. function `submitSocializingRewardsMerkleRoot`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L255-L267

```solidity
        if ((submissionCount == trustedNodesCount / 2 + 1)) {
            address socializingPool = IPoolUtils(staderConfig.getPoolUtils()).getSocializingPoolAddress(
                _rewardsData.poolId
            );
            ISocializingPool(socializingPool).handleRewards(_rewardsData);

            emit SocializingRewardsMerkleRootUpdated(
                _rewardsData.index,
                _rewardsData.merkleRoot,
                _rewardsData.poolId,
                block.number
            );
        }
```
3. function `submitSDPrice`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L290-L297

```solidity
        if ((submissionCount == (2 * trustedNodesCount) / 3 + 1)) {
            lastReportedSDPriceData = _sdPriceData;
            lastReportedSDPriceData.sdPriceInETH = getMedianValue(sdPrices);
            delete sdPrices;

            // Emit SD Price updated event
            emit SDPriceUpdated(_sdPriceData.sdPriceInETH, _sdPriceData.reportingBlockNumber, block.number);
        }
```
4. function `submitValidatorStats`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L371-L388

```solidity
        if (
            submissionCount == trustedNodesCount / 2 + 1 &&
            _validatorStats.reportingBlockNumber > validatorStats.reportingBlockNumber
        ) {
            validatorStats = _validatorStats;

            // Emit stats updated event
            emit ValidatorStatsUpdated(
                _validatorStats.reportingBlockNumber,
                _validatorStats.exitingValidatorsBalance,
                _validatorStats.exitedValidatorsBalance,
                _validatorStats.slashedValidatorsBalance,
                _validatorStats.exitingValidatorsCount,
                _validatorStats.exitedValidatorsCount,
                _validatorStats.slashedValidatorsCount,
                block.timestamp
            );
        }
```
5. function `submitWithdrawnValidators`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L431-L445

```solidity
        if (
            submissionCount == trustedNodesCount / 2 + 1 &&
            _withdrawnValidators.reportingBlockNumber > reportingBlockNumberForWithdrawnValidators
        ) {
            reportingBlockNumberForWithdrawnValidators = _withdrawnValidators.reportingBlockNumber;
            INodeRegistry(_withdrawnValidators.nodeRegistry).withdrawnValidators(_withdrawnValidators.sortedPubkeys);

            // Emit withdrawn validators updated event
            emit WithdrawnValidatorsUpdated(
                _withdrawnValidators.reportingBlockNumber,
                _withdrawnValidators.nodeRegistry,
                _withdrawnValidators.sortedPubkeys,
                block.timestamp
            );
        }
```
6. function `submitMissedAttestationPenalties`

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/StaderOracle.sol#L482-L493

```solidity
        if ((submissionCount == trustedNodesCount / 2 + 1)) {
            lastReportedMAPDIndex = _mapd.index;
            uint256 keyCount = _mapd.sortedPubkeys.length;
            for (uint256 i; i < keyCount; ) {
                bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_mapd.sortedPubkeys[i]);
                missedAttestationPenalty[pubkeyRoot]++;
                unchecked {
                    ++i;
                }
            }
            emit MissedAttestationPenaltyUpdated(_mapd.index, block.number, _mapd.sortedPubkeys);
        }
```
## Too Harsh Deactivation Rule on Operators
It only requires one front run validator as a culprit to deactivate an operator, as is evidenced in the following code logics of:

1. PermissionedNodeRegistry.sol with functions `allocateValidatorsAndUpdateOperatorId`, and `getTotalQueuedValidatorCount` getting denied:

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L670-L677

```solidity
    // handle front run validator by changing their status and deactivating operator
    function handleFrontRun(uint256 _validatorId) internal {
        validatorRegistry[_validatorId].status = ValidatorStatus.FRONT_RUN;
        uint256 operatorId = validatorRegistry[_validatorId].operatorId;
        if (operatorStructById[operatorId].active) {
            _deactivateNodeOperator(operatorId);
        }
    }
```
2. PermissionlessNodeRegistry.sol with functions `addValidatorKeys`, changeSocializingPoolState, and `updateOperatorDetails` getting denied:

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionlessNodeRegistry.sol#L624-L629

```solidity
    // handle front run validator by changing their status, deactivating operator and imposing penalty
    function handleFrontRun(uint256 _validatorId) internal {
        validatorRegistry[_validatorId].status = ValidatorStatus.FRONT_RUN;
        uint256 operatorId = validatorRegistry[_validatorId].operatorId;
        operatorStructById[operatorId].active = false;
    }
```
Consider enforcing this rule only when exceeding a threshold percentage of validators front running. 

## Typo mistakes
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L46

```diff
-     * @notice calculates the count of validator to deposit on beacon chain for a pool based on target weight and supply
+     * @notice calculates the count of validators to deposit on beacon chain for a pool based on target weight and supply
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol#L97

```diff
-        uint256 collateralETH = getCollateralETH(poolId, staderConfig); // 0, incase of permissioned NOs
+        uint256 collateralETH = getCollateralETH(poolId, staderConfig); // 0, in case of permissioned NOs
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L170

```diff
-     * @notice transfer the excess ETH sent by some EAO or non stader contract back to SSPM
+     * @notice transfer the excess ETH sent by some EOA or non stader contract back to SSPM
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L520

```diff
-     * @param _endIndex  up to end index of validator queue to to count
+     * @param _endIndex  up to end index of validator queue to count
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L104

```diff
-     * @return feeRecipientAddress fee recipient address for all validator clients of a operator
+     * @return feeRecipientAddress fee recipient address for all validator clients of an operator
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L38

```diff
-    /**route all call to this proxy contract to the respective latest vault contract
+    /**route all calls to this proxy contract to the respective latest vault contract
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol#L64

```diff
-     * @notice @update the owner of vault proxy contrat
+     * @notice @update the owner of vault proxy contract
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol#L32

```diff
-     * @dev pool index start from 1 with permission less pool
+     * @dev pool index starts from 1 with permissionless pool
```
## Repeatedly Used Instances May be Cached
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L106-L131

```diff
    function onboardNodeOperator(string calldata _operatorName, address payable _operatorRewardAddress)
        external
        override
        whenNotPaused
        returns (address feeRecipientAddress)
    {
        address poolUtils = staderConfig.getPoolUtils();
+        IPoolUtils _poolUtils = IPoolUtils(poolUtils);
-        if (IPoolUtils(poolUtils).poolAddressById(POOL_ID) != staderConfig.getPermissionedPool()) {
+        if (_poolUtils.poolAddressById(POOL_ID) != staderConfig.getPermissionedPool()) {
            revert DuplicatePoolIDOrPoolNotAdded();
        }
-        IPoolUtils(poolUtils).onlyValidName(_operatorName);
+        _poolUtils.onlyValidName(_operatorName);
        UtilLib.checkNonZeroAddress(_operatorRewardAddress);
        if (nextOperatorId > maxOperatorId) {
            revert MaxOperatorLimitReached();
        }
        if (!permissionList[msg.sender]) {
            revert NotAPermissionedNodeOperator();
        }
        //checks if operator already onboarded in any pool of protocol
-        if (IPoolUtils(poolUtils).isExistingOperator(msg.sender)) {
+        if (_poolUtils.isExistingOperator(msg.sender)) {
            revert OperatorAlreadyOnBoardedInProtocol();
        }
        feeRecipientAddress = staderConfig.getPermissionedSocializingPool();
        onboardOperator(_operatorName, _operatorRewardAddress);
        return feeRecipientAddress;
    }
```
## Essential Early Checks
Key checks should be placed at the beginning part of a function logic where possible:

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L106-L131

```diff
    function onboardNodeOperator(string calldata _operatorName, address payable _operatorRewardAddress)
        external
        override
        whenNotPaused
        returns (address feeRecipientAddress)
    {
+        if (!permissionList[msg.sender]) {
+            revert NotAPermissionedNodeOperator();
+        }
        address poolUtils = staderConfig.getPoolUtils();
        if (IPoolUtils(poolUtils).poolAddressById(POOL_ID) != staderConfig.getPermissionedPool()) {
            revert DuplicatePoolIDOrPoolNotAdded();
        }
        IPoolUtils(poolUtils).onlyValidName(_operatorName);
        UtilLib.checkNonZeroAddress(_operatorRewardAddress);
        if (nextOperatorId > maxOperatorId) {
            revert MaxOperatorLimitReached();
        }
-        if (!permissionList[msg.sender]) {
-            revert NotAPermissionedNodeOperator();
-        }
        //checks if operator already onboarded in any pool of protocol
        if (IPoolUtils(poolUtils).isExistingOperator(msg.sender)) {
            revert OperatorAlreadyOnBoardedInProtocol();
        }
        feeRecipientAddress = staderConfig.getPermissionedSocializingPool();
        onboardOperator(_operatorName, _operatorRewardAddress);
        return feeRecipientAddress;
    }
```
## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.16",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.