## 1. _msgSender() should be used instead of msg.sender in subcontracts of Context
*Instances*:
```
src/kuma-protocol/KBCToken.sol
24:         if (msg.sender != _KUMAAddressProvider.getKUMASwap(riskCategory)) {
101:         if (!IAccessControl(_KUMAAddressProvider.getAccessController()).hasRole(Roles.KUMA_MANAGER_ROLE, msg.sender)) {
102:             revert Errors.ACCESS_CONTROL_ACCOUNT_IS_MISSING_ROLE(msg.sender, Roles.KUMA_MANAGER_ROLE);

src/kuma-protocol/KIBToken.sol
43:         if (!_KUMAAddressProvider.getAccessController().hasRole(role, msg.sender)) {
44:             revert Errors.ACCESS_CONTROL_ACCOUNT_IS_MISSING_ROLE(msg.sender, role);

src/kuma-protocol/KUMAAccessController.sol
9:         _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
10:         _grantRole(Roles.KUMA_MANAGER_ROLE, msg.sender);

src/kuma-protocol/KUMAFeeCollector.sol
26:         if (!_KUMAAddressProvider.getAccessController().hasRole(Roles.KUMA_MANAGER_ROLE, msg.sender)) {
27:             revert Errors.ACCESS_CONTROL_ACCOUNT_IS_MISSING_ROLE(msg.sender, Roles.KUMA_MANAGER_ROLE);

src/kuma-protocol/KUMASwap.sol
58:         if (!IAccessControl(_KUMAAddressProvider.getAccessController()).hasRole(role, msg.sender)) {
59:             revert Errors.ACCESS_CONTROL_ACCOUNT_IS_MISSING_ROLE(msg.sender, role);
166:         KIBToken.mint(msg.sender, mintAmount);
167:         KUMABondToken.safeTransferFrom(msg.sender, address(this), tokenId);
170:         emit BondSold(tokenId, mintAmount, msg.sender);
212:                 msg.sender,
224:         KIBToken.burn(msg.sender, realizedBondValue);
227:             KUMABondToken.safeTransferFrom(address(this), msg.sender, tokenId);
230:         emit BondBought(tokenId, realizedBondValue, msg.sender);
286:         IKUMABondToken(KUMAAddressProvider.getKUMABondToken()).safeTransferFrom(address(this), msg.sender, tokenId);
311:         KIBToken.burn(msg.sender, amount);
312:         deprecationStableCoin.safeTransfer(msg.sender, redeemAmount);
314:         emit KIBTRedeemed(msg.sender, redeemAmount);

src/mcag-contracts/AccessController.sol
9:         _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
10:         _grantRole(Roles.MCAG_MINT_ROLE, msg.sender);
11:         _grantRole(Roles.MCAG_BURN_ROLE, msg.sender);
12:         _grantRole(Roles.MCAG_BLACKLIST_ROLE, msg.sender);
13:         _grantRole(Roles.MCAG_PAUSE_ROLE, msg.sender);
14:         _grantRole(Roles.MCAG_UNPAUSE_ROLE, msg.sender);
15:         _grantRole(Roles.MCAG_TRANSMITTER_ROLE, msg.sender);
16:         _grantRole(Roles.MCAG_MANAGER_ROLE, msg.sender);
17:         _grantRole(Roles.MCAG_SET_URI_ROLE, msg.sender);

src/mcag-contracts/KUMABondToken.sol
27:         if (!accessController.hasRole(role, msg.sender)) {
28:             revert Errors.ACCESS_CONTROL_ACCOUNT_IS_MISSING_ROLE(msg.sender, role);
148:         notBlacklisted(msg.sender)
173:         notBlacklisted(msg.sender)
189:         notBlacklisted(msg.sender)
209:         notBlacklisted(msg.sender)

src/mcag-contracts/KYCToken.sol
25:         if (!accessController.hasRole(role, msg.sender)) {
26:             revert Errors.ACCESS_CONTROL_ACCOUNT_IS_MISSING_ROLE(msg.sender, role);
```

## 2. Lack of validation on important parameters
Some important parameters of the contract interface need to be validated. Otherwise, the contract may not work properly due to the inadvertence or operation error of the caller, and even cause serious problems such as economic losses

[KUMABondToken.issueBond(address to, Bond calldata bond)](https://github.com/code-423n4/2023-02-kuma/blob/3f3d2269fcb3437a9f00ffdd67b5029487435b95/src/mcag-contracts/KUMABondToken.sol#L64): the bond data should be validated, like the values of `term`, `issuance`, `maturity`, `coupon`, `principal` should have some limitations, and `riskCategory` should be correctly calculated.

[KYCToken.mint(address to, KYCData calldata kycData)](https://github.com/code-423n4/2023-02-kuma/blob/3f3d2269fcb3437a9f00ffdd67b5029487435b95/src/mcag-contracts/KYCToken.sol#L49): the kycData should be validated, `kycData.owner` should not be 0, and `kycData.kycInfo` should not be 0.

## 3. Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (44)*:
```solidity
File: src/kuma-protocol/interfaces/IKBCToken.sol
8:     event KUMAAddressProviderSet(address KUMAAddressProvider);
9:     event CloneBondIssued(uint256 ghostId, CloneBond cloneBond);
10:     event CloneBondRedeemed(uint256 ghostId, uint256 parentId);
```
```solidity
File: src/kuma-protocol/interfaces/IKIBToken.sol
12:     event CumulativeYieldUpdated(uint256 oldCumulativeYield, uint256 newCumulativeYield);
13:     event EpochLengthSet(uint256 previousEpochLength, uint256 newEpochLength);
14:     event KUMAAddressProviderSet(address KUMAAddressProvider);
15:     event PreviousEpochCumulativeYieldUpdated(uint256 oldCumulativeYield, uint256 newCumulativeYield);
16:     event RiskCategorySet(bytes32 riskCategory);
17:     event YieldUpdated(uint256 oldYield, uint256 newYield);
```
```solidity
File: src/kuma-protocol/interfaces/IKUMAAddressProvider.sol
7:     event AccessControllerSet(address accessController);
8:     event KBCTokenSet(address KBCToken);
10:     event KUMABondTokenSet(address KUMABondToken);
15:     event RateFeedSet(address rateFeed);
```
```solidity
File: src/kuma-protocol/interfaces/IKUMAFeeCollector.sol
7:     event PayeeAdded(address indexed payee, uint256 share);
9:     event FeeReleased(uint256 income);
10:     event KUMAAddressProviderSet(address KUMAAddressProvider);
11:     event ShareUpdated(address indexed payee, uint256 newShare);
12:     event RiskCategorySet(bytes32 riskCategory);
```
```solidity
File: src/kuma-protocol/interfaces/IKUMASwap.sol
9:     event BondBought(uint256 tokenId, uint256 KIBTokenBurned, address indexed buyer);
10:     event BondClaimed(uint256 tokenId, uint256 cloneTokenId);
11:     event BondExpired(uint256 tokenId);
12:     event BondSold(uint256 tokenId, uint256 KIBTokenMinted, address indexed seller);
16:     event DeprecationStableCoinSet(address oldDeprecationStableCoin, address newDeprecationStableCoin);
17:     event FeeCharged(uint256 fee);
18:     event FeeSet(uint16 variableFee, uint256 fixedFee);
19:     event KIBTRedeemed(address indexed redeemer, uint256 redeemedStableCoinAmount);
20:     event KUMAAddressProviderSet(address KUMAAddressProvider);
21:     event MaxCouponsSet(uint256 maxCoupons);
22:     event MinCouponUpdated(uint256 oldMinCoupon, uint256 newMinCoupon);
23:     event RiskCategorySet(bytes32 riskCategory);
```
```solidity
File: src/kuma-protocol/interfaces/IMCAGRateFeed.sol
8:     event AccessControllerSet(address accessController);
```
```solidity
File: src/mcag-contracts/interfaces/IBlacklist.sol
7:     event AccessControllerSet(address accesController);
```
```solidity
File: src/mcag-contracts/interfaces/IKUMABondToken.sol
9:     event AccessControllerSet(address accesController);
10:     event BlacklistSet(address blacklist);
11:     event BondIssued(uint256 id, Bond bond);
12:     event BondRedeemed(uint256 id);
13:     event UriSet(string oldUri, string newUri);
```
```solidity
File: src/mcag-contracts/interfaces/IKYCToken.sol
7:     event AccessControllerSet(address accesController);
8:     event Mint(address indexed to, KYCData kycData);
9:     event Burn(uint256 tokenId, KYCData kycData);
10:     event UriSet(string oldUri, string newUri);
```
```solidity
File: src/mcag-contracts/interfaces/MCAGAggregatorInterface.sol
5:     event AccessControllerSet(address accesController);
6:     event AnswerTransmitted(address indexed transmitter, uint80 roundId, int256 answer);
7:     event MaxAnswerSet(int256 oldMaxAnswer, int256 newMaxAnswer);
```

## 4. Functions not used internally could be marked external

*Instances (10)*:
```solidity
File: src/kuma-protocol/KIBToken.sol
252:     function balanceOf(address account) public view override(ERC20Upgradeable, IERC20Upgradeable) returns (uint256) {
259:     function totalSupply() public view override(ERC20Upgradeable, IERC20Upgradeable) returns (uint256) {
```
```solidity
File: src/mcag-contracts/KUMABondToken.sol
143:     function approve(address to, uint256 tokenId)
169:     function setApprovalForAll(address operator, bool approved)
185:     function transferFrom(address from, address to, uint256 tokenId)
205:     function safeTransferFrom(address from, address to, uint256 tokenId, bytes memory data)
```
```solidity
File: src/mcag-contracts/KYCToken.sol
105:     function approve(address to, uint256 tokenId) public pure override(ERC721, IERC721) {
112:     function setApprovalForAll(address operator, bool approved) public pure override(ERC721, IERC721) {
119:     function transferFrom(address from, address to, uint256 tokenId) public pure override(ERC721, IERC721) {
126:     function safeTransferFrom(address from, address to, uint256 tokenId, bytes memory data)
```

