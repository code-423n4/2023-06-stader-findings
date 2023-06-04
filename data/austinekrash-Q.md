Import declarations should import specific identifiers, rather than the whole file
Using import declarations of the form import {<identifier_name>} from "some/file.sol" avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation.

Make the changes in all contracts
