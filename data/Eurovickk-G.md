https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol

In the getPubkeyRoot function, instead of checking the length of _pubkey and reverting if it doesn't match VALIDATOR_PUBKEY_LENGTH, you can use a require statement at the beginning of the function to check the length. This can save gas by combining the length check and the revert into a single operation.

function getPubkeyRoot(bytes memory _pubkey) public pure returns (bytes32) {
    require(_pubkey.length == VALIDATOR_PUBKEY_LENGTH, "Invalid pubkey length");

    Rest of the function...
}

In this updated version, we use the require statement at the beginning of the function to check the length of the _pubkey parameter. If the length doesn't match VALIDATOR_PUBKEY_LENGTH, which is likely a predefined constant, it will revert with the specified error message. This combines the length check and the revert into a single operation, saving gas.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/VaultProxy.sol

In the fallback function, consider using the staticcall instead of delegatecall if you only need to read data from the vault implementation. The staticcall is cheaper in terms of gas compared to delegatecall, which allows modifying contract state. 

fallback(bytes calldata _input) external payable returns (bytes memory) {
    address vaultImplementation = isValidatorWithdrawalVault
        ? staderConfig.getValidatorWithdrawalVaultImplementation()
        : staderConfig.getNodeELRewardVaultImplementation();

    // Use staticcall instead of delegatecall if only reading data
    (bool success, bytes memory data) = vaultImplementation.staticcall(_input);

    if (!success) {
        revert(string(data));
    }

    return data;
}


fallback(bytes calldata _input) external payable returns (bytes memory) {
    address vaultImplementation = isValidatorWithdrawalVault
        ? staderConfig.getValidatorWithdrawalVaultImplementation()
        : staderConfig.getNodeELRewardVaultImplementation();

    // Use staticcall instead of delegatecall if only reading data
    (bool success, bytes memory data) = vaultImplementation.staticcall(_input);

    if (!success) {
        revert(string(data));
    }

    return data;
}
By using staticcall, the contract only reads data from the vault implementation and avoids the overhead of modifying contract state, resulting in lower gas costs.


In the updateOwner function, you can remove the UtilLib.checkNonZeroAddress(_owner) call as the onlyOwner modifier already ensures that the _owner address is non-zero. This optimization avoids redundant address checks.


function updateOwner(address _owner) external override onlyOwner {
    owner = _owner;
    emit UpdatedOwner(owner);
}

In this updated version, the UtilLib.checkNonZeroAddress(_owner) call is removed because the onlyOwner modifier already ensures that the _owner address is non-zero. This optimization avoids redundant address checks while maintaining the necessary validation.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/ValidatorWithdrawalVault.sol

In the calculateValidatorWithdrawalShare function, you can replace the if conditions checking contractBalance with usersETH and TOTAL_STAKED_ETH using Math.min() to simplify the logic. For example: contractBalance = Math.min(contractBalance, TOTAL_STAKED_ETH);. 
In the calculateValidatorWithdrawalShare function, you can optimize the assignment of rewards by combining the addition operations into a single line.

function calculateValidatorWithdrawalShare()
    public
    view
    returns (
        uint256 _userShare,
        uint256 _operatorShare,
        uint256 _protocolShare
    )
{
    uint8 poolId = VaultProxy(payable(address(this))).poolId();
    IStaderConfig staderConfig = VaultProxy(payable(address(this))).staderConfig();
    uint256 TOTAL_STAKED_ETH = staderConfig.getStakedEthPerNode();
    uint256 collateralETH = getCollateralETH(poolId, staderConfig); // 0, incase of permissioned NOs
    uint256 usersETH = TOTAL_STAKED_ETH - collateralETH;
    uint256 contractBalance = address(this).balance;

    contractBalance = Math.min(contractBalance, TOTAL_STAKED_ETH);

    uint256 totalRewards = contractBalance - TOTAL_STAKED_ETH;
    _operatorShare = collateralETH;
    _userShare = usersETH;

    if (totalRewards > 0) {
        (uint256 userReward, uint256 operatorReward, uint256 protocolReward) = IPoolUtils(
            staderConfig.getPoolUtils()
        ).calculateRewardShare(poolId, totalRewards);
        (_userShare, _operatorShare, _protocolShare) = (
            _userShare + userReward,
            _operatorShare + operatorReward,
            _protocolShare + protocolReward
        );
    }
}


https://github.com/code-423n4/2023-06-stader/blob/main/contracts/OperatorRewardsCollector.sol


Consider using the payable modifier for the depositFund and reimburseUserFund functions. This modifier explicitly marks the functions as capable of receiving Ether and can provide a slight gas optimization. In the withdrawFund function, instead of calling payable(msg.sender).call{value: _amount}('') to transfer funds, you can use payable(msg.sender).transfer(_amount), which is a more concise and gas-efficient way to transfer Ether to the designated recipient.

pragma solidity 0.8.16;

import './library/UtilLib.sol';

import './interfaces/IStaderConfig.sol';
import './interfaces/IPermissionedPool.sol';
import './interfaces/IStaderInsuranceFund.sol';

import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';
import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

contract StaderInsuranceFund is
    IStaderInsuranceFund,
    Initializable,
    AccessControlUpgradeable,
    ReentrancyGuardUpgradeable
{
    IStaderConfig public staderConfig;

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

    function initialize(address _admin, address _staderConfig) public initializer {
        UtilLib.checkNonZeroAddress(_admin);
        UtilLib.checkNonZeroAddress(_staderConfig);
        __AccessControl_init_unchained();
        __ReentrancyGuard_init();
        staderConfig = IStaderConfig(_staderConfig);
        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
    }

    // function to add fund for insurance
    receive() external payable {
        emit ReceivedInsuranceFund(msg.value);
    }

    // `MANAGER` can withdraw access fund
    function withdrawFund(uint256 _amount) external override nonReentrant {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        if (address(this).balance < _amount || _amount == 0) {
            revert InvalidAmountProvided();
        }

        //slither-disable-next-line arbitrary-send-eth
        payable(msg.sender).transfer(_amount);
        emit FundWithdrawn(_amount);
    }

    /**
     * @notice reimburse 1 ETH per key to SSPM for permissioned NOs doing front running or giving invalid signature
     * @dev only permissioned pool can call
     * @param _amount amount of ETH to transfer to permissioned pool
     */
    function reimburseUserFund(uint256 _amount) external override nonReentrant payable {
        UtilLib.onlyStaderContract(msg.sender, staderConfig, staderConfig.PERMISSIONED_POOL());
        if (address(this).balance < _amount) {
            revert InSufficientBalance();
        }
        IPermissionedPool(staderConfig.getPermissionedPool()).receiveInsuranceFund{value: _amount}();
    }

    //update the address of staderConfig
    function updateStaderConfig(address _staderConfig) external onlyRole(DEFAULT_ADMIN_ROLE) {
        UtilLib.checkNonZeroAddress(_staderConfig);
        staderConfig = IStaderConfig(_staderConfig);
        emit UpdatedStaderConfig(_staderConfig);
    }
}

In the updated code:

The depositFund function now uses the receive modifier, which allows the contract to receive Ether without the need for an explicit function call.
The reimburseUserFund function now includes the payable modifier, explicitly allowing it to receive Ether.
The withdrawFund function uses payable(msg.sender).transfer(_amount) to transfer funds, which is a more concise and gas-efficient method compared to payable(msg.sender).call{value: _amount}('').
These optimizations help improve the gas efficiency of the contract.

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PoolSelector.sol

Consider using the uint8 data type instead of uint256 for the pool weights. Since the POOL_WEIGHTS_SUM is defined as 10000, it can be represented by a uint16 instead of a uint256. This change can help reduce gas costs when storing and manipulating these values. 
In the computePoolAllocationForDeposit function, the Math.min function is called multiple times. Instead of calling it repeatedly, you can store the intermediate result in a variable and use it in subsequent calculations to avoid redundant function calls. Review the use of storage operations within loops. For example, in the poolAllocationForExcessETHDeposit function, there are multiple storage read and write operations within the loop. Consider batching or optimizing these operations to reduce gas costs. 
Carefully review the use of division and multiplication operations, as they can be expensive in terms of gas. Look for opportunities to minimize or simplify these operations, if possible. Consider using the view modifier for functions that don't modify contract state. This can provide additional gas optimizations by indicating that the function only reads data and does not change the contract's state.

uint8 public constant POOL_WEIGHTS_SUM = 100;
mapping(uint8 => uint256) public poolWeights;

Storing the intermediate result of Math.min in the computePoolAllocationForDeposit function:

uint256 minCapacity = Math.min(poolAllocationMaxSize, _newValidatorToRegister);
selectedPoolCapacity = Math.min(minCapacity, Math.min(remainingPoolCapacity, remainingPoolTarget));

Optimizing storage operations in the poolAllocationForExcessETHDeposit function:

uint8[] memory poolIdArray = IPoolUtils(staderConfig.getPoolUtils()).getPoolIdArray();
uint256 poolCount = poolIdArray.length;
uint256 ETH_PER_NODE = staderConfig.getStakedEthPerNode();
selectedPoolCapacity = new uint256[](poolCount);
uint256 selectedValidatorCount;
uint256 i = poolIdArrayIndexForExcessDeposit;

uint256 poolCapacity;
uint256 poolDepositSize;
uint256 remainingValidatorsToDeposit;

for (uint256 j; j < poolCount; j++) {
    poolCapacity = poolUtils.getQueuedValidatorCountByPool(poolIdArray[i]);
    poolDepositSize = ETH_PER_NODE - poolUtils.getCollateralETH(poolIdArray[i]);
    remainingValidatorsToDeposit = ethToDeposit / poolDepositSize;

    selectedPoolCapacity[i] = Math.min(poolAllocationMaxSize - selectedValidatorCount, Math.min(poolCapacity, remainingValidatorsToDeposit));
    selectedValidatorCount += selectedPoolCapacity[i];
    ethToDeposit -= selectedPoolCapacity[i] * poolDepositSize;
    i = (i + 1) % poolCount;

    if (ethToDeposit < ETH_PER_NODE || selectedValidatorCount >= poolAllocationMaxSize) {
        poolIdArrayIndexForExcessDeposit = i;
        break;
    }
}

Using the view modifier for functions that don't modify contract state:

function computePoolAllocationForDeposit(uint8 _poolId, uint256 _newValidatorToRegister) external view override returns (uint256 selectedPoolCapacity) {
    // Function logic
}

function updatePoolWeights(uint256[] calldata _poolTargets) external {
    // Function logic
}

function updatePoolAllocationMaxSize(uint16 _poolAllocationMaxSize) external {
    // Function logic
}

https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol

Gas optimization: Gas optimization is important for efficient contract execution. In the updateTotalPenaltyAmount function, the loop variable i is incremented using the unchecked keyword. However, since the loop condition is i < reportedValidatorCount, an unchecked increment is not necessary. You can remove the unchecked keyword.

function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
    uint256 reportedValidatorCount = _pubkey.length;
    for (uint256 i = 0; i < reportedValidatorCount; i++) {
        if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
            revert ValidatorSettled();
        }
        bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
        // Retrieve the penalty for changing the fee recipient address based on Rated.network data.
        uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
        uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);

        // Compute the total penalty for the given validator public key,
        // taking into account additional penalties and penalty reversals from the DAO.
        uint256 totalPenalty = _mevTheftPenalty + _missedAttestationPenalty + additionalPenaltyAmount[pubkeyRoot];
        totalPenaltyAmount[_pubkey[i]] = totalPenalty;
        if (totalPenalty >= validatorExitPenaltyThreshold) {
            emit ForceExitValidator(_pubkey[i]);
        }
    }
}

Gas optimization: In the updateTotalPenaltyAmount function, you are using the totalPenalty variable only for checking if it exceeds the validatorExitPenaltyThreshold. You can move the if (totalPenalty >= validatorExitPenaltyThreshold) condition inside the loop and emit the ForceExitValidator event directly when the total penalty exceeds the threshold, without storing the value in the totalPenaltyAmount mapping.

function updateTotalPenaltyAmount(bytes[] calldata _pubkey) external override nonReentrant {
    uint256 reportedValidatorCount = _pubkey.length;
    for (uint256 i = 0; i < reportedValidatorCount; i++) {
        if (UtilLib.getValidatorSettleStatus(_pubkey[i], staderConfig)) {
            revert ValidatorSettled();
        }
        bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
        // Retrieve the penalty for changing the fee recipient address based on Rated.network data.
        uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
        uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);

        // Compute the total penalty for the given validator public key,
        // taking into account additional penalties and penalty reversals from the DAO.
        uint256 totalPenalty = _mevTheftPenalty + _missedAttestationPenalty + additionalPenaltyAmount[pubkeyRoot];
        if (totalPenalty >= validatorExitPenaltyThreshold) {
            emit ForceExitValidator(_pubkey[i]);
        }
        totalPenaltyAmount[_pubkey[i]] = totalPenalty;
    }
}

Gas optimization: In the calculateMEVTheftPenalty function, you are using the violatedEpochs.length to calculate the penalty. Instead of using an array, you can use the .length property of the violatedEpochs array directly to avoid unnecessary memory allocation.

function calculateMEVTheftPenalty(bytes32 _pubkeyRoot) public view override returns (uint256) {
    // Retrieve the epochs in which the validator violated the fee recipient change rule.
    uint256 violatedEpochsCount = IRatedV1(ratedOracleAddress).getViolationsForValidator(_pubkeyRoot).length;

    // each strike attracts `mevTheftPenaltyPerStrike` penalty
    return violatedEpochsCount * mevTheftPenaltyPerStrike;
}

Gas optimization and security: In the updateTotalPenaltyAmount function, you are checking if the validator has already settled using the UtilLib.getValidatorSettleStatus function. It's important to ensure that this function correctly handles potential storage slot collisions and cannot be manipulated by malicious actors.


https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Auction.sol

In the createLot function, consider moving the lotItem storage variable declaration to the beginning of the function to reduce gas costs.
In the addBid function, consider using require instead of if conditions followed by a revert statement for better gas optimization.
In the claimSD function, the AlreadyClaimed modifier could be combined with the previous AuctionNotEnded modifier to reduce redundant checks and improve gas efficiency.
Similarly, review other functions in the contract for potential gas optimizations, such as reducing redundant state checks or eliminating unnecessary operations.

// SPDX-License-Identifier: MIT
pragma solidity 0.8.16;

import './library/UtilLib.sol';

import '../contracts/interfaces/SDCollateral/IAuction.sol';
import '../contracts/interfaces/IStaderStakePoolManager.sol';

import '@openzeppelin/contracts/token/ERC20/IERC20.sol';
import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';
import '@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol';
import '@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol';

contract Auction is IAuction, Initializable, AccessControlUpgradeable, PausableUpgradeable, ReentrancyGuardUpgradeable {
    struct LotItem {
        uint256 startBlock;
        uint256 endBlock;
        uint256 sdAmount;
        uint256 highestBidAmount;
        address highestBidder;
        bool sdClaimed;
        bool ethExtracted;
        mapping(address => uint256) bids;
    }

    IStaderConfig public override staderConfig;
    uint256 public override nextLot;
    uint256 public override bidIncrement;
    uint256 public override duration;

    mapping(uint256 => LotItem) public lots;

    uint256 public constant MIN_AUCTION_DURATION = 7200; // 24 hours

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

    function initialize(address _admin, address _staderConfig) external initializer {
        UtilLib.checkNonZeroAddress(_admin);
        UtilLib.checkNonZeroAddress(_staderConfig);

        __AccessControl_init();
        __Pausable_init();
        __ReentrancyGuard_init();

        staderConfig = IStaderConfig(_staderConfig);
        duration = 2 * MIN_AUCTION_DURATION;
        bidIncrement = 5e15;
        nextLot = 1;

        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
        emit UpdatedStaderConfig(_staderConfig);
        emit AuctionDurationUpdated(duration);
        emit BidIncrementUpdated(bidIncrement);
    }

    function createLot(uint256 _sdAmount) external override whenNotPaused {
        LotItem storage lotItem = lots[nextLot]; // Move the lotItem storage variable declaration to the beginning

        lotItem.startBlock = block.number;
        lotItem.endBlock = block.number + duration;
        lotItem.sdAmount = _sdAmount;

        if (!IERC20(staderConfig.getStaderToken()).transferFrom(msg.sender, address(this), _sdAmount)) {
            revert SDTransferFailed();
        }
        emit LotCreated(nextLot, lotItem.sdAmount, lotItem.startBlock, lotItem.endBlock, bidIncrement);
        nextLot++;
    }

    function addBid(uint256 lotId) external payable override whenNotPaused {
        // reject payments of 0 ETH
        require(msg.value > 0, "InSufficientETH"); // Use require statement for better gas optimization

        LotItem storage lotItem = lots[lotId];
        require(block.number <= lotItem.endBlock, "AuctionEnded"); // Use require statement for better gas optimization

        uint256 totalUserBid = lotItem.bids[msg.sender] + msg.value;

        require(totalUserBid >= lotItem.highestBidAmount + bidIncrement, "InSufficientBid"); // Use require statement for better gas optimization

        lotItem.highestBidder = msg.sender;
        lotItem.highestBidAmount = totalUserBid;
        lotItem.bids[msg.sender] = totalUserBid;

        emit BidPlaced