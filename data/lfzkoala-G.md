# Issue: Inefficient Usage of Gas (Gas Optimization)

## Description:
In the `createLot` function, there is unnecessary code redundancy which can lead to inefficient usage of gas. The code assigns values to the lots mapping multiple times, when it could simply do it once.

## Location:
2023-06-stader/contracts/Auction.sol#67
```
function createLot(uint256 _sdAmount) external override whenNotPaused {
    ...
    lots[nextLot].startBlock = block.number;
    lots[nextLot].endBlock = block.number + duration;
    lots[nextLot].sdAmount = _sdAmount;
    ...
}
```
## Suggestion:
Assign all the values to the lots mapping at once to optimize gas usage.

```
function createLot(uint256 _sdAmount) external override whenNotPaused {
    ...
    lots[nextLot] = LotItem({
        startBlock: block.number,
        endBlock: block.number + duration,
        sdAmount: _sdAmount,
        ...
    });
    ...
}
```
# Issue Name: Gas Optimization for `getValidatorsByOperator`

## Description of the security issue: 
If a contract call is expensive in terms of gas costs, it could potentially block operations because the call consumes all the gas provided for the transaction.
Proof of Concept of Attack with code example: The `getValidatorsByOperator` function could potentially be expensive when `_pageSize` is set to a high value, potentially causing the transaction to fail due to out of gas errors.

## Location of the security issue with code snippet:

```
function getValidatorsByOperator(
    address _operator,
    uint256 _pageNumber,
    uint256 _pageSize
) external view override returns (Validator[] memory) {
    ...
    for (uint256 i = startIndex; i < endIndex; i++) {
        uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
        validators[i - startIndex] = validatorRegistry[validatorId];
    }
    return validators;
}
```

## How to resolve the security issue with code example: 
Consider setting an upper limit to `_pageSize` to prevent high gas cost operations. If an upper limit is not feasible, perhaps consider a different method of retrieving the validators, such as a function that returns one validator at a time.

```
function getValidatorsByOperator(
    address _operator,
    uint256 _pageNumber,
    uint256 _pageSize
) external view override returns (Validator[] memory) {
    require(_pageSize <= MAX_PAGE_SIZE, "Page size too large");
    ...
}
```





