L-01: whitelistPermissionedNOs() is missing the method to set to false
The current permissionList[] can only be set to true, and there is no method to cancel it, so if the setting is wrong, it cannot be cancelled.
Suggested changes to pass in the status
```solidity
-   function whitelistPermissionedNOs(address[] calldata _permissionedNOs) external override {
+   function whitelistPermissionedNOs(address[] calldata _permissionedNOs,bool isPermission) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        uint256 permissionedNosLength = _permissionedNOs.length;
        for (uint256 i = 0; i < permissionedNosLength; i++) {
            address operator = _permissionedNOs[i];
            UtilLib.checkNonZeroAddress(operator);
-           permissionList[operator] = true;
+           permissionList[operator] = isPermission;                        
            emit OperatorWhitelisted(operator);
        }
    }
```


L-02: updatePoolThreshold() is missing the check that _units.length > 0 

The current protocol `_units.length == 0`, will be treated as if `poolThreshold` is not set
Suggest adding check

```solidity
    function updatePoolThreshold(
        uint8 _poolId,
        uint256 _minThreshold,
        uint256 _maxThreshold,
        uint256 _withdrawThreshold,
        string memory _units
    ) external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        if ((_minThreshold > _withdrawThreshold) || (_minThreshold > _maxThreshold)) {
            revert InvalidPoolLimit();
        }
+       require(bytes(_units).length >0,"length ==0");
```


L-03: getMinimumSDToBond() has the risk of precision loss

`getMinimumSDToBond()` currently performs `convertETHToSD(poolThreshold.minThreshold)`, this will have precision loss, if `minThreshold` is small
It is recommended to multiply `_numValidator` before conversion

```solidity
    function getMinimumSDToBond(uint8 _poolId, uint256 _numValidator)
        public
        view
        override
        returns (uint256 _minSDToBond)
    {
        isPoolThresholdValid(_poolId);
        PoolThresholdInfo storage poolThreshold = poolThresholdbyPoolId[_poolId];


-       _minSDToBond = convertETHToSD(poolThreshold.minThreshold);
-       _minSDToBond *= _numValidator;
+       _minSDToBond = convertETHToSD(poolThreshold.minThreshold * _numValidator);
    }
```

L-04: updateAdmin() is missing to determine whether the old admin is the same as the new one, which may cause the new admin to be _revokeRole

`updateAdmin()` will set the new admin and then revoke the old one.
Currently there is no restriction that the old and new cannot be the same, and the new one is set first and then the old one is cancelled, so if it is the same it will end up without any admin

```solidity
    function updateAdmin(address _admin) external onlyRole(DEFAULT_ADMIN_ROLE) {
        address oldAdmin = accountsMap[ADMIN];
+       require(oldAdmin!=_admin,"same");
        _grantRole(DEFAULT_ADMIN_ROLE, _admin);
        setAccount(ADMIN, _admin);
        _revokeRole(DEFAULT_ADMIN_ROLE, oldAdmin);
    }
```