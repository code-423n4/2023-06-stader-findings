Import declarations should import specific identifiers, rather than the whole file
Using import declarations of the form import {<identifier_name>} from "some/file.sol" avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation.

Make the changes in all contracts


contracts\PermissionedNodeRegistry.sol:
x += y (x -= y) costs more gas than x = x + y (x = x - y) for state variables
280: preDepositValidatorCount += _count;
285: preDepositValidatorCount -= _c
