##### 1. Aution.sol
  - **Incorrect Validation of Auction Duration**
    - [Links to affected code](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L144)
    - **Description**: The updateDuration function in the Auction contract does not properly validate the _duration parameter to ensure it falls within an acceptable range. This can lead to potential issues where caller can set a duration that is too large, causing the auction to remain unsettled indefinitely.
    - **Impact**: If an invalid duration value is set, it can result in the auction not being settled and potentially causing delays in the auction process.
    - **Recommended Mitigation Steps**: To address this issue, it is recommended to add proper validation to ensure the _duration parameter falls within an acceptable range. Here is a possible solution of how the code can be improved:
    
    ```solidity
    function updateDuration(uint256 _duration) external override {
      UtilLib.onlyManagerRole(msg.sender, staderConfig);
      if (_duration < MIN_AUCTION_DURATION || _duration > MAX_AUCTION_DURATION) {
          revert InvalidDuration();
      }
      duration = _duration;
      emit AuctionDurationUpdated(duration);
    }
    ```
- **Missing Validation for Bid Increment**
  - [Links to affected code](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L151)
  - **Description**: The updateBidIncrement function in the Auction contract does not check if the _bidIncrement parameter is equal to zero. This can lead to issues where the bid increment is unintentionally set to zero, causing the auction to become unresponsive.
  - **Impact**: If the _bidIncrement parameter is set to zero, it can prevent any further bidding in the auction, rendering it ineffective.
  - **Recommended Mitigation Steps**: To mitigate this issue, it is advisable to include a validation check within the updateBidIncrement function to ensure that the _bidIncrement parameter is not zero. Here's a solution of how the code can be updated:

    ```solidity
      function updateBidIncrement(uint256 _bidIncrement) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        require(_bidIncrement > 0, "Bid increment must be greater than zero");
        bidIncrement = _bidIncrement;
        emit BidIncrementUpdated(_bidIncrement);
      }
    ```
##### 2. PermissionedNodeRegistry.sol
  - **Missing Negative Check for _pageNumber in getAllActiveValidators and getValidatorsByOperator functions**
  - [Links to affexted code](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#LL590C4-L590C4)
  - **Description**: Those two function in the contract does not include a check to ensure that the provided _pageNumber is not negative.
  - **Impact**: The missing check for a negative _pageNumber can lead to unexpected behavior or reverts if a negative _pageNumber is provided as input. This can potentially disrupt the intended functionality of the contract and cause inconvenience or loss for users.
  - **Recommended Mitigation Steps**: 
    - Add a check to ensure that the _pageNumber is greater than 0 before proceeding with the function execution.
    - Handle the case of a negative _pageNumber by reverting the function call with an appropriate error message.
    Example Fix:
    ```solidity
    function getAllActiveValidators(uint256 _pageNumber, uint256 _pageSize) external view override
        returns (Validator[] memory)
      {
          require(_pageNumber > 0, "InvalidPageNumber"); // Add negative check
          uint256 startIndex = (_pageNumber - 1) * _pageSize + 1;
          uint256 endIndex = startIndex + _pageSize;
          endIndex = endIndex > nextValidatorId ? nextValidatorId : endIndex;
          Validator[] memory validators = new Validator[](_pageSize);
          uint256 validatorCount = 0;
          for (uint256 i = startIndex; i < endIndex; i++) {
              if (isActiveValidator(i)) {
                  validators[validatorCount] = validatorRegistry[i];
                  validatorCount++;
              }
          }
          assembly {
              mstore(validators, validatorCount)
          }
          return validators;
      }
    ```
  - **Add parentheses in Boolean Condition in isExistingPubkey and isExistingOperator Functions to improve readability**
  - [Links to affexted code](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedNodeRegistry.sol#L647)
  - **Description**: The isExistingPubkey and isExistingOperator functions in the contract have boolean conditions as return without parentheses around them. While this doesn't affect the functionality of the code, it can reduce code readability and make it slightly more difficult to understand the intended logic.
  - **Impact**: Worse readability
  - **Recommended Mitigation Steps**: Add parentheses around the boolean conditions to improve code readability and explicitly convey the intended logic.
    ```solidity
    function isExistingPubkey(bytes calldata _pubkey) external view override returns (bool) {
        return (validatorIdByPubkey[_pubkey] != 0);
    }

    function isExistingOperator(address _operAddr) external view override returns (bool) {
        return (operatorIDByAddress[_operAddr] != 0);
    }
    ```
##### 3. PermissionedPool.sol
  - **Potential Integer Overflow in decreasePreDepositValidatorCount Function**
  - [Links to affexted code](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/PermissionedPool.sol#L283)
  - **Description**: The decreasePreDepositValidatorCount function in the codebase has a potential issue that could lead to an integer overflow, resulting in the preDepositValidatorCount variable becoming a negative value. This could occur if the decreaseAmount parameter passed to the function is larger than the current value of preDepositValidatorCount.
  - **Impact**: If the decreasePreDepositValidatorCount function results in a negative value for preDepositValidatorCount, it could have adverse effects on the system's functionality. This may cause unexpected behavior in other parts of the codebase that rely on the preDepositValidatorCount variable, potentially leading to incorrect calculations, data inconsistencies, or even system crashes.
  - **Recommended Mitigation Steps**: To mitigate this bug and prevent the integer overflow, we can use the following changes:
    ```solidity
    function decreasePreDepositValidatorCount(uint256 decreaseAmount) internal {
      require(preDepositValidatorCount >= decreaseAmount, "Insufficient preDepositValidatorCount");
      preDepositValidatorCount -= decreaseAmount;
    }

    ```