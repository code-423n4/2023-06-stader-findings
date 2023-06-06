







[G‑01]	Reduce gas usage by moving to Solidity 0.8.19 or later	47	-
[G‑02]	Avoid updating storage when the value hasn't changed	21	16800
[G‑03]	Remove or replace unused state variables	1	-
[G‑04]	Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate	2	-
[G‑05]	State variables can be packed into fewer storage slots	2	-
[G‑06]	Structs can be packed into fewer storage slots	2	-
[G‑07]	Using storage instead of memory for structs/arrays saves gas	1	4200
[G‑08]	State variables should be cached in stack variables rather than re-reading them from storage	181	17557
[G‑09]	Multiple accesses of a mapping/array should use a local variable cache	18	756
[G‑10]	The result of function calls should be cached rather than re-calling the function	12	-
[G‑11]	<x> += <y> costs more gas than <x> = <x> + <y> for state variables	12	1356
[G‑12]	internal functions only called once can be inlined to save gas	32	640
[G‑13]	Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement	1	85
[G‑14]	++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops	16	960
[G‑15]	Optimize names to save gas	38	836
[G‑16]	Using bools for storage incurs overhead	5	85500
[G‑17]	>= costs less gas than >	7	21
[G‑18]	++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)	36	180
[G‑19]	Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead	6	-
[G‑20]	Using private rather than public for constants, saves gas	35	-
[G‑21]	Don't use SafeMath once the solidity version is 0.8.0 or greater	2	-
[G‑22]	Division by two should use bit shifting	6	120
[G‑23]	Superfluous event fields	13	-
[G‑24]	Functions guaranteed to revert when called by normal users can be marked payable	71	1491
[G‑25]	Constructors can be marked payable	21	441
[G‑26]	Not using the named return variables anywhere in the function is confusing	7	-