## Kuma Protocol Gas Optimization Report

This submission serves as a gas optimization report of the Kuma Protocol codebase. The ways the codebase can be optimized are as follows:

## `KBCToken.sol`

### KBC-01G: Variable Data Location Optimization (Affected Lines: [L51](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KBCToken.sol#L51))

The `cBond` argument of the `KBCToken::issueBond` function is set as `memory` whilst the function itself is `external`. We advise it to be set to `calldata` optimizing the `issueBond` function's execution cost in `KUMASwap::buyBond`.

### KBC-02G: Inefficient Variable Declaration (Affected Lines: [L72](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KBCToken.sol#L72))

The `KBCToken::redeem` function loads the full `CloneBond` struct into memory only to utilize the `cBond.parentId` member of it. We advise the `_bonds[tokenId].parentId` entry to be stored to a `uint256` variable that is consequently utilized, optimizing the function's gas cost significantly.

## `KIBToken.sol` 

### KIB-01G: Unused Private Contract Member (Affected Lines: [L40](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L40))

The referenced `_allowances` mapping present in the `KIBToken` implementation remains unused in the codebase. We advise it to be safely omitted as its presence can cause ambiguity with the homonym `_allowances` mapping of the `ERC20Upgradeable` implementation of OpenZeppelin.

### KIB-02G: Redundant External Self Calls (Affected Lines: [L142](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L142), [L170](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L170), [L276](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L276), [L281](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L281))

The referenced statements perform an external self-call as they specify `this.balanceOf` instead of `balanceOf` directly, instructing the compiler to treat the statement as an external call to the `address(this)` target. We advise the code to utilize `balanceOf` directly, avoiding the significant gas overhead of external calls. If the code wishes to be verbose about which implementation it invokes, the `this.balanceOf` statements can be replaced by `KIBToken.balanceOf`, instructing the compiler to invoke the `KIBToken` implementation of `balanceOf` explicitly.

### KIB-03G: Potential Arithmetic Optimizations (Affected Lines: [L175](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L175), [L280](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L280), [L367](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L367))

The referenced arithmetic calculations are guaranteed to be performed safely by the conditional clauses whose execution precedes them. We advise them to be wrapped in `unchecked` code blocks, optimizing their execution. We should note that more calculations can theoretically be performed in `unchecked` code blocks (i.e. `block.timestamp - _lastRefresh` in `KIBToken::_calculateCumulativeYield`), however, the referenced lines are **guaranteed by conditionals rather than logic** to be performed safely and thus their validity is infallible.

## `KUMAAddressProvider.sol`

### KAP-01G: Redundant `Initializable` Import & Inheritence (Affected Lines: [L14](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAAddressProvider.sol#L14))

The `Initializable` contract is part of the `UUPSUpgradeable` inheritance chain and is also imported and inherited directly by `KUMAAddressProvider`. We advise it to be safely omitted from the contract's import list and direct inheritance declarations as it is ineffectual.

## `KUMAFeeCollector.sol`

### KFC-01G: Redundant `Initializable` Import & Inheritence (Affected Lines: [L14](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L14))

The `Initializable` contract is part of the `UUPSUpgradeable` inheritance chain and is also imported and inherited directly by `KUMAFeeCollector`. We advise it to be safely omitted from the contract's import list and direct inheritance declarations as it is ineffectual.

### KFC-02G: Potential Iterator Optimizations (Affected Lines: [L165](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L165), [L174](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L174), [L208](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L208))

The referenced iterator increments / decrements are performed "safely" via Solidity's built-in safe arithmetic due to the usage of the `0.8.17` version compiler. We advise the relevant increment / decrement statements to be relocated to the end of their respective `for` loop and wrapped in an `unchecked` code block, optimizing their execution.

### KFC-03G: Inefficient Loop Bounds (Affected Lines: [L165-L166](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L165-L166))

The referenced loop iterates from `payeesLength` to `1`, however, on each iteration it utilizes the `i` iterator by subtracting it by `1`. We advise the iteration to start from `payeesLength - 1`, apply no conditional for the loop, and contain an `if (i == 0) break;` statement at the end of the `for` loop, significantly optimizing the loop's gas cost.

### KFC-04G: Inefficient Adjustment of Storage Variable (Affected Lines: [L185](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L185))

The `_totalShares` variable is updated on each iteration of the `newPayees` array instead of being assigned to once at the end of the `KUMAFeeCollector::changePayees` function's execution. We advise a local `totalShares` / `totalShares_` variable to be declared that is incremented on each iteration of the `newPayees` array. Finally, we advise the `_totalShares` variable to be updated once after the `for` loop with the value of `totalShares` / `totalShares_`, requiring a single `SSTORE` operation for the function's execution.

### KFC-05G: Potential Arithmetic Optimizations (Affected Lines: [L136](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L136), [L138](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L138))

The referenced arithmetic calculations are guaranteed to be performed safely by the conditional clauses whose execution precedes them. We advise them to be wrapped in `unchecked` code blocks, optimizing their execution. We should note that more calculations can theoretically be performed in `unchecked` code blocks (i.e. `_totalShares -= _shares[payee]` in `KUMAFeeCollector::removePayee`), however, the referenced lines are **guaranteed by conditionals rather than logic** to be performed safely and thus their validity is infallible.

### KFC-06G: Inefficient Loop Limit Evaluation (Affected Lines: [L208](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L208))

As loop limits in Solidity are dynamically evaluated when they contain function calls, the referenced `length` invocation will be repeated on each iteration of the `for` loop. We advise the `length` to be cached to a local variable that is consequently utilized for the `for` loop as it is not expected to change during the `KUMAFeeCollector::_release` function's execution.

## `KUMASwap.sol`

### KSP-01G: Unused Named Function Arguments (Affected Lines: [L561](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMASwap.sol#L561))

The `KUMASwap::onERC721Received` function contains named function arguments yet does not utilize them. We advise their explicit names to be omitted. To note, a function signature is defined by the data types and not the variable names. As such, a `function` declaration of `onERC721Received(address, address, uint256, bytes calldata)` is entirely valid.

### KSP-02G: Inefficient Loop Limit Evaluation (Affected Lines: [L614](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMASwap.sol#L614))

As loop limits in Solidity are dynamically evaluated when they contain function calls, the referenced `length` invocation will be repeated on each iteration of the `for` loop. We advise the `length` to be cached to a local variable that is consequently utilized for the `for` loop as it is not expected to change during the `KUMASwap::_updateMinCoupon` function's execution.

### KSP-03G: Potential Arithmetic Optimizations (Affected Lines: [L587](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMASwap.sol#L587))

The referenced arithmetic calculations are guaranteed to be performed safely by the conditional clauses whose execution precedes them. We advise them to be wrapped in `unchecked` code blocks, optimizing their execution. We should note that more calculations can theoretically be performed in `unchecked` code blocks (i.e. `_couponInventory[bond.coupon]` in `KUMASwap::buyBond`), however, the referenced lines are **guaranteed by conditionals rather than logic** to be performed safely and thus their validity is infallible.

## `KYCToken.sol`

### KYC-01G: Non-Standard Override of Transfer Functionality (Affected Lines: [L119-L121](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KYCToken.sol#L119-L121), [L126-L132](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KYCToken.sol#L126-L132))

The `KYCToken` is meant to represent a non-transferrable token and this is achieved by overriding the relevant `transferFrom` and `safeTransferFrom` functions of the `ERC721` dependency of OpenZeppelin. However, the current mechanism only overrides `transferFrom(address, address, uint256)` and `safeTransferFrom(address, address, uint256, bytes memory)` and not `safeTransferFrom(address, address, uint256)`.

The `safeTransferFrom(address, address, uint256)` function invokes `safeTransferFrom(address, address, uint256, bytes memory)` in the code of OpenZeppelin and is protected, however, the current way of preventing transfers is non-standard and non-uniform. We advise the `_beforeTokenTransfer` hook to be overridden instead, ensuring that regardless of the implementations of `ERC721` the code will never permit a transfer as the `_beforeTokenTransfer` hook is guaranteed to be executed in all transfer flows including mint and burn operations.

## `KUMABondToken.sol`

### KBT-01G: Non-Standard Override of Transfer Functionality (Affected Lines: [L185-L197](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KUMABondToken.sol#L185-L197), [L205-L217](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KUMABondToken.sol#L205-L217))

The `KUMABondToken` is meant to represent a blacklist-able and pausable token and this is achieved by overriding the relevant `transferFrom` and `safeTransferFrom` functions of the `ERC721` dependency of OpenZeppelin. However, the current mechanism only overrides `transferFrom(address, address, uint256)` and `safeTransferFrom(address, address, uint256, bytes memory)` and not `safeTransferFrom(address, address, uint256)`.

The `safeTransferFrom(address, address, uint256)` function invokes `safeTransferFrom(address, address, uint256, bytes memory)` in the code of OpenZeppelin and is protected, however, the current way of preventing transfers is non-standard and non-uniform. We advise the `_beforeTokenTransfer` hook to be overridden instead, ensuring that regardless of the implementations of `ERC721` the code will apply proper access control to transfers as the `_beforeTokenTransfer` hook is guaranteed to be executed in all transfer flows including mint and burn operations.

### KBT-02G: Inconsistent Usage of Caller / `_msgSender` (Affected Lines: [L148](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KUMABondToken.sol#L148), [L156](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KUMABondToken.sol#L156), [L173](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KUMABondToken.sol#L173), [L176](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KUMABondToken.sol#L176), [L189](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KUMABondToken.sol#L189), [L193](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KUMABondToken.sol#L193), [L209](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KUMABondToken.sol#L209), [L213](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KUMABondToken.sol#L213))

The codebase of `KUMABondToken` applies modifier-based access control on transfers and approvals by using the `msg.sender` value whilst the code of the implementations make use of the `_msgSender` GSN-compliant method. We advise the style to be standardized to either `msg.sender` or `_msgSender` across all instances, the former of which will lead to a gas optimization as it will not incur the gas overhead of invoking an internal function. To note, this does not constitute a vulnerability as the `_msgSender` implementation of the OpenZeppelin dependency in use yields the `msg.sender` value.

## `MCAGRateFeed.sol`

### MRF-01G: Redundant `Initializable` Import & Inheritence (Affected Lines: [L13](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/MCAGRateFeed.sol#L13))

The `Initializable` contract is part of the `UUPSUpgradeable` inheritance chain and is also imported and inherited directly by `MCAGRateFeed`. We advise it to be safely omitted from the contract's import list and direct inheritance declarations as it is ineffectual.

### MRF-02G: Conditional Optimization (Affected Lines: [L92](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/MCAGRateFeed.sol#L92))

The referenced conditional within `MCAGRateFeed::getRate` will evaluate whether a `rate` is below the `_MIN_RATE_COUPON` threshold and yield the value of `_MIN_RATE_COUPON` instead in such a case. We advise the conditional to be made inclusive, optimizing the code's gas cost in the case of `rate == _MIN_RATE_COUPON`. This optimization is possible as inclusive and non-inclusive comparators utilize the same gas thus reducing the code's gas cost by one `JUMP` instruction if `rate == _MIN_RATE_COUPON`.

### MRF-03G: Potential Arithmetic Optimizations (Affected Lines: [L87](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/MCAGRateFeed.sol#L87), [L89](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/MCAGRateFeed.sol#L89))

The referenced arithmetic calculations are guaranteed to be performed safely by the conditional clauses whose execution precedes them. We advise them to be wrapped in `unchecked` code blocks, optimizing their execution. The referenced lines are **guaranteed by conditionals rather than logic** to be performed safely and thus their validity is infallible.

## `MCAGAggregator.sol`

### MAR-01G: Improperly Empty Chainlink Format Variable (Affected Lines: [L86](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/MCAGAggregator.sol#L86))

The `MCAGAggregator::latestRoundData` function is meant to conform to the Chainlink paradigm and while it does fill in both the `roundId` and `answeredInRound` variables (the latter being irrelevant to the Kuma Protocol), the function does not fill in the `startedAt` value. We advise the `startedAt` value to be set to the same value as `updatedAt`, replicating the Chainlink paradigm and ensuring users of the `latestRoundData` can apply the same sanitizations regardless of whether they integrate with a Chainlink oracle or with the `MCAGAggregator`.