# Overview
This QA report addresses two key issues within the Stader protocol's smart contract codebase. The first pertains to informational naming and consistency within the Auction contract. The second relates to the ordering of functions and naming of internal functions within the SDCollateral contract. Each of these areas has been identified as requiring attention to align with best practices and improve code readability and maintainability.

#Informational Naming
Within the Auction contract, there is an inconsistency in the capitalization of an error message in the [claimSD](https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L83) function:
```
if (msg.sender != lotItem.highestBidder) revert notQualified();
```

The lowercase naming of notQualified does not match the usual practice of starting error messages with a capital letter. Therefore, it is recommended that this be updated to NotQualified for consistency and clarity:
```
if (msg.sender != lotItem.highestBidder) revert NotQualified(); 
```


# Order of Functions and naming internal function
The order of functions as per their visibility
The placement and naming of the slashSD internal function

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered:
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private

# SDCollateral slashSD internal function
In reviewing the SDCollateral contract, it was noted that the slashSD internal function does not adhere to the recommended order. This function is found within the block of external functions. Specifically, slashSD is situated between two external functions in the codebase, which can potentially cause confusion.

Here is the specific location of the slashSD function in the contract:
https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/SDCollateral.sol#LL90C14-L90C21

## Internal Function Naming
It is considered a good practice to prefix the names of internal and private functions with an underscore (_). This convention allows developers to immediately recognize the intended visibility of a function, improving clarity and reducing the likelihood of inadvertent visibility errors.


# Recommendations
For maintaining clarity and standard code organization practices, it is recommended that the slashSD function be relocated to the internal functions section of the contract. This change will align the contract with the recommended order and promote better readability and maintainability of the contract code.

By implementing this improvement, the contract would be easier to understand, thus reducing the potential for misunderstanding and bugs. This will also facilitate future code reviews and audits.

function **_slashSD**(address _operator, uint256 _sdToSlash) **internal** {


Here is an example how OpenZeppelin name the internal function:
```
function _transfer(address from, address to, uint256 amount) internal {
```
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/e29e42a529dafcdb5d792cfef97ce71ea0388045/contracts/token/ERC20/ERC20Upgradeable.sol#L222

# Conclusion
Implementing these changes will enhance the clarity and maintainability of the Stader protocol's smart contract code. By aligning with best practices, these improvements will also facilitate future code reviews and audits, ultimately contributing to the robustness and reliability of the protocol.