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