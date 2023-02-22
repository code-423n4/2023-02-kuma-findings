## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 36 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 2 |
| [GAS-3](#GAS-3) | Cache array length outside of loop | 3 |
| [GAS-4](#GAS-4) | State variables should be cached in stack variables rather than re-reading them from storage | 3 |
| [GAS-5](#GAS-5) | Use calldata instead of memory for function arguments that do not get mutated | 13 |
| [GAS-6](#GAS-6) | Functions guaranteed to revert when called by normal users can be marked `payable` | 31 |
| [GAS-7](#GAS-7) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 3 |
| [GAS-8](#GAS-8) | Using `private` rather than `public` for constants, saves gas | 5 |
| [GAS-9](#GAS-9) | Use != 0 instead of > 0 for unsigned integer comparison | 11 |
### <a name="GAS-1"></a>[GAS-1] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (36)*:
```solidity
File: src/kuma-protocol/KBCToken.sol

33:         if (address(KUMAAddressProvider) == address(0)) {

87:         if (_ownerOf(tokenId) == address(0)) {

```

```solidity
File: src/kuma-protocol/KIBToken.sol

71:         if (address(KUMAAddressProvider) == address(0)) {

74:         if (currency == bytes4(0) || country == bytes4(0) || term == 0) {

74:         if (currency == bytes4(0) || country == bytes4(0) || term == 0) {

136:         if (account == address(0)) {

164:         if (account == address(0)) {

267:         if (from == address(0)) {

270:         if (to == address(0)) {

```

```solidity
File: src/kuma-protocol/KUMAAddressProvider.sol

26:         if (_address == address(0)) {

42:         if (address(accessController) == address(0)) {

136:         if (currency == bytes4(0) || country == bytes4(0) || term == 0) {

136:         if (currency == bytes4(0) || country == bytes4(0) || term == 0) {

```

```solidity
File: src/kuma-protocol/KUMAFeeCollector.sol

39:         if (address(KUMAAddressProvider) == address(0)) {

42:         if (currency == bytes4(0) || country == bytes4(0) || term == 0) {

42:         if (currency == bytes4(0) || country == bytes4(0) || term == 0) {

81:         if (payee == address(0)) {

175:             if (newPayees[i] == address(0)) {

```

```solidity
File: src/kuma-protocol/KUMASwap.sol

93:         if (address(KUMAAddressProvider) == address(0) || address(deprecationStableCoin) == address(0)) {

93:         if (address(KUMAAddressProvider) == address(0) || address(deprecationStableCoin) == address(0)) {

96:         if (currency == bytes4(0) || country == bytes4(0) || term == 0) {

96:         if (currency == bytes4(0) || country == bytes4(0) || term == 0) {

251:         if (buyer == address(0)) {

375:         if (address(newDeprecationStableCoin) == address(0)) {

```

```solidity
File: src/kuma-protocol/MCAGRateFeed.sol

34:         if (address(accessController) == address(0)) {

54:         if (currency == bytes4(0) || country == bytes4(0) || term == 0) {

54:         if (currency == bytes4(0) || country == bytes4(0) || term == 0) {

57:         if (address(oracle) == address(0)) {

```

```solidity
File: src/mcag-contracts/Blacklist.sol

25:         if (address(_accessController) == address(0)) {

```

```solidity
File: src/mcag-contracts/KUMABondToken.sol

45:         if (address(_accessController) == address(0) || address(_blacklist) == address(0)) {

45:         if (address(_accessController) == address(0) || address(_blacklist) == address(0)) {

131:         if (_ownerOf(tokenId) == address(0)) {

```

```solidity
File: src/mcag-contracts/KYCToken.sol

35:         if (address(_accessController) == address(0)) {

64:         if (_ownerOf(tokenId) == address(0)) {

96:         if (_ownerOf(tokenId) == address(0)) {

```

```solidity
File: src/mcag-contracts/MCAGAggregator.sol

34:         if (address(_accessController) == address(0)) {

```

### <a name="GAS-2"></a>[GAS-2] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (2)*:
```solidity
File: src/kuma-protocol/KUMASwap.sol

35:     bool private _isDeprecated;

```

```solidity
File: src/mcag-contracts/Blacklist.sol

12:     mapping(address => bool) private _blacklisted;

```

### <a name="GAS-3"></a>[GAS-3] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (3)*:
```solidity
File: src/kuma-protocol/KUMAFeeCollector.sol

174:         for (uint256 i; i < newPayees.length; i++) {

208:         for (uint256 i; i < _payees.length(); i++) {

```

```solidity
File: src/kuma-protocol/KUMASwap.sol

614:         for (uint256 i = 1; i < _coupons.length();) {

```

### <a name="GAS-4"></a>[GAS-4] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (3)*:
```solidity
File: src/kuma-protocol/KIBToken.sol

327:         uint256 referenceRate = IMCAGRateFeed(_KUMAAddressProvider.getRateFeed()).getRate(_riskCategory);

```

```solidity
File: src/kuma-protocol/KUMASwap.sol

132:         uint256 referenceRate = IMCAGRateFeed(KUMAAddressProvider.getRateFeed()).getRate(_riskCategory);

```

```solidity
File: src/mcag-contracts/MCAGAggregator.sol

91:         answeredInRound = _roundId;

```

### <a name="GAS-5"></a>[GAS-5] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (13)*:
```solidity
File: src/kuma-protocol/KBCToken.sol

51:     function issueBond(address to, CloneBond memory cBond)

```

```solidity
File: src/kuma-protocol/KIBToken.sol

60:         string memory name,

61:         string memory symbol,

```

```solidity
File: src/kuma-protocol/interfaces/IKBCToken.sol

28:     function issueBond(address to, CloneBond memory cBond) external returns (uint256 tokenId);

```

```solidity
File: src/kuma-protocol/interfaces/IKIBToken.sol

20:         string memory name,

21:         string memory symbol,

```

```solidity
File: src/mcag-contracts/KUMABondToken.sol

100:     function setUri(string memory newUri) external override onlyRole(Roles.MCAG_SET_URI_ROLE) {

205:     function safeTransferFrom(address from, address to, uint256 tokenId, bytes memory data)

```

```solidity
File: src/mcag-contracts/KYCToken.sol

79:     function setUri(string memory newUri) external override onlyRole(Roles.MCAG_SET_URI_ROLE) {

126:     function safeTransferFrom(address from, address to, uint256 tokenId, bytes memory data)

```

```solidity
File: src/mcag-contracts/MCAGAggregator.sol

33:     constructor(string memory description_, int256 maxAnswer_, IAccessControl _accessController) {

```

```solidity
File: src/mcag-contracts/interfaces/IKUMABondToken.sol

44:     function setUri(string memory newUri) external;

```

```solidity
File: src/mcag-contracts/interfaces/IKYCToken.sol

21:     function setUri(string memory newUri) external;

```

### <a name="GAS-6"></a>[GAS-6] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (31)*:
```solidity
File: src/kuma-protocol/KBCToken.sol

71:     function redeem(uint256 tokenId) external override onlyKUMASwap(_bonds[tokenId].parentId) {

```

```solidity
File: src/kuma-protocol/KIBToken.sol

100:     function setEpochLength(uint256 epochLength) external override onlyRole(Roles.KUMA_SET_EPOCH_LENGTH_ROLE) {

371:     function _authorizeUpgrade(address newImplementation) internal view override onlyRole(Roles.KUMA_MANAGER_ROLE) {}

```

```solidity
File: src/kuma-protocol/KUMAAddressProvider.sol

50:     function setKBCToken(address KBCToken) external override onlyManager onlyValidAddress(KBCToken) {

55:     function setRateFeed(address rateFeed) external override onlyManager onlyValidAddress(rateFeed) {

60:     function setKUMABondToken(address KUMABondToken) external override onlyManager onlyValidAddress(KUMABondToken) {

142:     function _authorizeUpgrade(address newImplementation) internal override onlyManager {}

```

```solidity
File: src/kuma-protocol/KUMAFeeCollector.sol

77:     function addPayee(address payee, uint256 share) external override onlyManager {

103:     function removePayee(address payee) external override onlyManager {

123:     function updatePayeeShare(address payee, uint256 share) external onlyManager {

152:     function changePayees(address[] calldata newPayees, uint256[] calldata newShares) external override onlyManager {

249:     function _authorizeUpgrade(address newImplementation) internal override onlyManager {}

```

```solidity
File: src/kuma-protocol/KUMASwap.sol

342:     function pause() external override onlyRole(Roles.KUMA_SWAP_PAUSE_ROLE.toGranularRole(_riskCategory)) {

349:     function unpause() external override onlyRole(Roles.KUMA_SWAP_UNPAUSE_ROLE.toGranularRole(_riskCategory)) {

359:     function setFees(uint16 variableFee, uint256 fixedFee) external override onlyRole(Roles.KUMA_MANAGER_ROLE) {

385:     function initializeDeprecationMode() external override onlyRole(Roles.KUMA_MANAGER_ROLE) whenNotDeprecated {

398:     function uninitializeDeprecationMode() external onlyRole(Roles.KUMA_MANAGER_ROLE) whenNotDeprecated {

412:     function enableDeprecationMode() external override onlyRole(Roles.KUMA_MANAGER_ROLE) whenNotDeprecated {

570:     function _authorizeUpgrade(address newImplementation) internal view override onlyRole(Roles.KUMA_MANAGER_ROLE) {}

```

```solidity
File: src/kuma-protocol/MCAGRateFeed.sol

121:     function _authorizeUpgrade(address newImplementation) internal override onlyManager {}

```

```solidity
File: src/mcag-contracts/Blacklist.sol

37:     function blacklist(address account) external override onlyBlacklister {

46:     function unBlacklist(address account) external override onlyBlacklister {

```

```solidity
File: src/mcag-contracts/KUMABondToken.sol

86:     function redeem(uint256 tokenId) external override onlyRole(Roles.MCAG_BURN_ROLE) whenNotPaused {

100:     function setUri(string memory newUri) external override onlyRole(Roles.MCAG_SET_URI_ROLE) {

108:     function pause() external override onlyRole(Roles.MCAG_PAUSE_ROLE) {

115:     function unpause() external override onlyRole(Roles.MCAG_UNPAUSE_ROLE) {

```

```solidity
File: src/mcag-contracts/KYCToken.sol

49:     function mint(address to, KYCData calldata kycData) external override onlyRole(Roles.MCAG_MINT_ROLE) {

63:     function burn(uint256 tokenId) external override onlyRole(Roles.MCAG_BURN_ROLE) {

79:     function setUri(string memory newUri) external override onlyRole(Roles.MCAG_SET_URI_ROLE) {

```

```solidity
File: src/mcag-contracts/MCAGAggregator.sol

52:     function transmit(int256 answer) external override onlyRole(Roles.MCAG_TRANSMITTER_ROLE) {

69:     function setMaxAnswer(int256 newMaxAnswer) external onlyRole(Roles.MCAG_MANAGER_ROLE) {

```

### <a name="GAS-7"></a>[GAS-7] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (3)*:
```solidity
File: src/kuma-protocol/KUMAFeeCollector.sol

174:         for (uint256 i; i < newPayees.length; i++) {

208:         for (uint256 i; i < _payees.length(); i++) {

```

```solidity
File: src/kuma-protocol/KUMASwap.sol

150:         _couponInventory[bond.coupon]++;

```

### <a name="GAS-8"></a>[GAS-8] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (5)*:
```solidity
File: src/kuma-protocol/KIBToken.sol

25:     uint256 public constant MAX_YIELD = 1e29;

26:     uint256 public constant MAX_EPOCH_LENGTH = 365 days;

27:     uint256 public constant MIN_YIELD = WadRayMath.RAY;

```

```solidity
File: src/kuma-protocol/KUMASwap.sol

29:     uint256 public constant MIN_ALLOWED_COUPON = WadRayMath.RAY;

30:     uint256 public constant DEPRECATION_MODE_TIMELOCK = 2 days;

```

### <a name="GAS-9"></a>[GAS-9] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (11)*:
```solidity
File: src/kuma-protocol/KIBToken.sol

145:         if (amount > 0) {

177:         if (amount > 0) {

287:         if (amount > 0) {

```

```solidity
File: src/kuma-protocol/KUMAFeeCollector.sol

164:         if (payeesLength > 0) {

165:             for (uint256 i = payeesLength; i > 0; i--) {

223:         if (availableIncome > 0) {

```

```solidity
File: src/kuma-protocol/KUMASwap.sol

161:         if (fee > 0) {

188:         if (_expiredBonds.length() > 0 && !isBondExpired) {

548:         return _expiredBonds.length() > 0;

637:         if (_variableFee > 0) {

640:         if (_fixedFee > 0) {

```
