L-01
changePayees() Suggest adding to check whether newPayees are duplicated to avoid _totalShares error

```solidity
    function changePayees(address[] calldata newPayees, uint256[] calldata newShares) external override onlyManager {
...

        for (uint256 i; i < newPayees.length; i++) {
+	    if (_payees.contains(newPayees[i])) {
+	          revert Errors.PAYEE_ALREADY_EXISTS();
+	    }    

            if (newPayees[i] == address(0)) {
                revert Errors.CANNOT_SET_TO_ADDRESS_ZERO();
            }
            if (newShares[i] == 0) {
                revert Errors.SHARE_CANNOT_BE_ZERO();
            }

            address payee = newPayees[i];
            _payees.add(payee);
            _shares[payee] = newShares[i];
            _totalShares += newShares[i];

            emit PayeeAdded(payee, newShares[i]);
        }
   }

```
L-02
setFees() Suggest adding a size limit
variableFee is the percentage, it is recommended to check that it does not exceed 100% i.e. 1e4

```solidity
contract KUMASwap is IKUMASwap, PausableUpgradeable, UUPSUpgradeable {
...
    function setFees(uint16 variableFee, uint256 fixedFee) external override onlyRole(Roles.KUMA_MANAGER_ROLE) {
+       require(variableFee < 1e4,"variableFee invalid");
        _variableFee = variableFee;
        _fixedFee = fixedFee;
        emit FeeSet(variableFee, fixedFee);
    }
```

L-03
KUMABondToken.issueBond() Suggest adding a check whether riskCategory is correct
riskCategory is used in many places, especially as a key in addressProvider, it is recommended that issueBond check it
```solidity
contract KUMABondToken is ERC721, Pausable, IKUMABondToken {
...

   function issueBond(address to, Bond calldata bond)
        external
        override
        onlyRole(Roles.MCAG_MINT_ROLE)
        notBlacklisted(to)
        whenNotPaused
    {
+   	require(bond.riskCategory==keccak256(abi.encode(bond.currency, bond.country, bond.term)));
        _tokenIdCounter.increment();
        uint256 tokenId = _tokenIdCounter.current();
        _bonds[tokenId] = bond; 
        _safeMint(to, tokenId);
        emit BondIssued(tokenId, bond);
    }
```

L-04
setDeprecationStableCoin() suggests adding the restriction _deprecationInitializedAt needs to be equal to 0
Before entering Deprecated, there will be a buffer time to give the user the right and time to choose, such as whether to buyBond() or sellBond()
When enter Deprecated, can only buyBondForStableCoin()/redeemKIBT() both methods are using StableCoin
So it is important to use which StableCoin, To avoid changing the StableCoin before the buffer time expires into Deprecated
It is recommended that the StableCoin addresss should not be modified after the proposed Deprecated.

```solidity
    function setDeprecationStableCoin(IERC20 newDeprecationStableCoin)
        external
        override
        onlyRole(Roles.KUMA_MANAGER_ROLE)
        whenNotDeprecated
    {
...
+     require(_deprecationInitializedAt == 0,"_deprecationInitializedAt don't zero");
        if (address(newDeprecationStableCoin) == address(0)) {
            revert Errors.CANNOT_SET_TO_ADDRESS_ZERO();
        }
        emit DeprecationStableCoinSet(address(_deprecationStableCoin), address(newDeprecationStableCoin));
        _deprecationStableCoin = newDeprecationStableCoin;  
```


L-05
setEpochLength() may still cause _previousEpochCumulativeYield to be smaller than the previous one
In order to avoid the previousEpochTimestamp being shifted back after resetting epochLength, we added the check
if epochLength > _epochLength then refreshed and then resetting epochLength

```solidity
    function setEpochLength(uint256 epochLength) external override onlyRole(Roles.KUMA_SET_EPOCH_LENGTH_ROLE) {
        if (epochLength == 0) {
            revert Errors.EPOCH_LENGTH_CANNOT_BE_ZERO();
        }
        if (epochLength > MAX_EPOCH_LENGTH) {
            revert Errors.NEW_EPOCH_LENGTH_TOO_HIGH();
        }     
        if (epochLength > _epochLength) {  //<--------refresh
            _refreshCumulativeYield();
            _refreshYield();
        }
        emit EpochLengthSet(_epochLength, epochLength);
        _epochLength = epochLength;
    }
```

But epochLength < _epochLength is also possible for the previousEpochTimestamp to be shifted back, because the time is calculated by taking the modulus.
So it is recommended to remove the epochLength > _epochLength limit:

```solidity
    function setEpochLength(uint256 epochLength) external override onlyRole(Roles.KUMA_SET_EPOCH_LENGTH_ROLE) {
        if (epochLength == 0) {
            revert Errors.EPOCH_LENGTH_CANNOT_BE_ZERO();
        }
        if (epochLength > MAX_EPOCH_LENGTH) {
            revert Errors.NEW_EPOCH_LENGTH_TOO_HIGH();
        }     
-       if (epochLength > _epochLength) {
            _refreshCumulativeYield();
            _refreshYield();
-       }
        emit EpochLengthSet(_epochLength, epochLength);
        _epochLength = epochLength;
    }
```