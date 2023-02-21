## Kuma Protocol Q&A Report

This submission serves as a Q&A report of the Kuma Protocol codebase. The findings that were identified with a low / non-critical security impact are as follows:

## `KBCToken.sol`

### KBC-01L: Improper Disable of Initializer (Affected Lines: [L30](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KBCToken.sol#L30))

The Kuma Protocol repository makes use of the `v4.8.1` dependency of OpenZeppelin with the `Initializer` contract being our focus. The contract contains a dedicated `_disableInitializers` function meant to be invoked by contract `constructor` implementations in contracts that inherit it, ensuring that the contract cannot be initialized **or re-initialized** in the future. 

In the current implementation, the `initializer` modifier is simply added to the contract's `constructor` which does not disable the initializer properly as it still permits **a re-initialization to occur**. In the current version of the codebase this does not pose an issue as a `reinitializer` modifier is not in use, however, in a future deployment this may manifest.

## `KIBToken.sol`

### KIB-01L: Improper Disable of Initializer (Affected Lines: [L49](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L49))

The Kuma Protocol repository makes use of the `v4.8.1` dependency of OpenZeppelin with the `Initializer` contract being our focus. The contract contains a dedicated `_disableInitializers` function meant to be invoked by contract `constructor` implementations in contracts that inherit it, ensuring that the contract cannot be initialized **or re-initialized** in the future. 

In the current implementation, the `initializer` modifier is simply added to the contract's `constructor` which does not disable the initializer properly as it still permits **a re-initialization to occur**. In the current version of the codebase this does not pose an issue as a `reinitializer` modifier is not in use, however, in a future deployment this may manifest.

### KIB-02L: Insufficient Initial Epoch Sanitization (Affected Lines: [L68](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L68))

While the `KIBToken::setEpochLength` function properly evaluates that the `epochLength` provided is not greater-than (`>`) the `MAX_EPOCH_LENGTH`, no such sanitization is applied during the `KIBToken::initialize` function. We advise the `initialize` function to apply the same logical checks as `setEpochLength`, potentially refactored to an `internal` function that both code segments utilize.

### KIB-03L: Discrepant Epoch Inclusivity Definitions (Affected Lines: [L79-L81](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L79-L81), [L342-L345](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L342-L345))

The `KIBToken::_getPreviousEpochTimestamp` and `KIBToken::initialize` functions differ in the way they define the "current" and "previous" epoch. In the `initialize` function, the `_lastRefresh` is either set to the current `block.timestamp` or `(block.timestamp / epochLength) * epochLength + epochLength`, while in the `_getPreviousEpochTimestamp` function the value yielded is either `block.timestamp` or `(block.timestamp / epochLength) * epochLength`. 

We advise either the assignment in `initialize` to add the `epochLength` to the current `block.timestamp` or the `_getPreviousEpochTimestamp` to subtract the `epochLength` from the `block.timestamp`, the former of which we consider standard.

### KIB-04L: Inexistent Enforcement of Minimums / Maximums in Yield (Affected Lines: [L331](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KIBToken.sol#L331))

The `MAX_YIELD` constant variable declared in `KIBToken` remains unutilized. We advise it to be enforced in the `_refreshYield` function by ensuring that the `lowestYield` being set is at most equal to `MAX_YIELD`, in which case the value of `MAX_YIELD` should be used instead. Additionally, the `MIN_YIELD` value should be enforced in a similar fashion.

## `KUMAAddressProvider.sol`

### KAP-01L: Improper Disable of Initializer (Affected Lines: [L39](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAAddressProvider.sol#L39))

The Kuma Protocol repository makes use of the `v4.8.1` dependency of OpenZeppelin with the `Initializer` contract being our focus. The contract contains a dedicated `_disableInitializers` function meant to be invoked by contract `constructor` implementations in contracts that inherit it, ensuring that the contract cannot be initialized **or re-initialized** in the future. 

In the current implementation, the `initializer` modifier is simply added to the contract's `constructor` which does not disable the initializer properly as it still permits **a re-initialization to occur**. In the current version of the codebase this does not pose an issue as a `reinitializer` modifier is not in use, however, in a future deployment this may manifest.

## `KUMAFeeCollector.sol`

### KFC-01L: Improper Disable of Initializer (Affected Lines: [L32](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L32))

The Kuma Protocol repository makes use of the `v4.8.1` dependency of OpenZeppelin with the `Initializer` contract being our focus. The contract contains a dedicated `_disableInitializers` function meant to be invoked by contract `constructor` implementations in contracts that inherit it, ensuring that the contract cannot be initialized **or re-initialized** in the future. 

In the current implementation, the `initializer` modifier is simply added to the contract's `constructor` which does not disable the initializer properly as it still permits **a re-initialization to occur**. In the current version of the codebase this does not pose an issue as a `reinitializer` modifier is not in use, however, in a future deployment this may manifest.

### KFC-02L: Improper Release Event (Affected Lines: [L88](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L88), [L160](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L160))

The `KUMAFeeCollector::addPayee` and `KUMAFeeCollector::changePayees` functions may operate on an empty `_payees` data entry. In such a case, if funds to-be distributed are present in the contract the `KUMAFeeCollector::_releaseIfAvailableIncome` function will incorrectly execute `KUMAFeeCollector::_release` which in turn will emit the `FeeReleased` event for the available income. We advise the `_releaseIfAvailableIncome` function to be invoked solely when `_payees.length()` is non-zero in the referenced instances as otherwise incorrect events may be emitted in `addPayee` and `changePayees`.

### KFC-03L: Inexistent Duplicate Entry Prevention (Affected Lines: [L175-L180](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMAFeeCollector.sol#L175-L180))

The `KUMAFeeCollector::changePayees` function does not adequately sanitize the new payees, permitting duplicate entries to exist which will cause the contract to significantly misbehave as it would track the `_totalShares` incorrectly, and perform two payouts with the latest `newShares[i]` value. We advise the code to add a new `if` conditional which causes the code to fail if `_payees.contains(newPayees[i])` evaluates to `true`.

## `KUMASwap.sol` 

### KSP-01L: Inexistent Limitation of Variable Fee (Affected Lines: [L360](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMASwap.sol#L360))

The `KUMASwap::setFees` function does not apply any sanitization on the `variableFee` value, permitting a fee to be applied that exceeds the 100% value of the bond (i.e. `PercentageMath::PERCENTAGE_FACTOR`). We advise the `variableFee` input variable of `setFees` to be validated as less than `PERCENTAGE_FACTOR` and preferably less than a stricter value (i.e. 20%) to ensure unfair fees are not applied in the protocol. 

This finding also ties in with the slippage-related vulnerability submitted in a separate exhibit whereby a user could submit a transaction with a blockchain state that applies a 5% fee and the fee could change between the transaction's submission and the transaction's execution by the network.

### KSP-02L: Inexistent Limitation of Bond Fee (Affected Lines: [L636-L643](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMASwap.sol#L636-L643))

The `KUMASwap::_calculateFees` function does not apply adequate sanitization to the `fee` that is ultimately applied to the `amount` which can potentially exceed it due to the presence of a `_fixedFee`. We advise the code to mandate that the final `fee` calculated does not exceed the `amount` supplied as an argument and to fail in such a case.

### KSP-03L: Improper Disable of Initializer (Affected Lines: [L78](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMASwap.sol#L78))

The Kuma Protocol repository makes use of the `v4.8.1` dependency of OpenZeppelin with the `Initializer` contract being our focus. The contract contains a dedicated `_disableInitializers` function meant to be invoked by contract `constructor` implementations in contracts that inherit it, ensuring that the contract cannot be initialized **or re-initialized** in the future. 

In the current implementation, the `initializer` modifier is simply added to the contract's `constructor` which does not disable the initializer properly as it still permits **a re-initialization to occur**. In the current version of the codebase this does not pose an issue as a `reinitializer` modifier is not in use, however, in a future deployment this may manifest.

### KSP-04L: Unsafe Casting of Term Months (Affected Lines: [L100](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/KUMASwap.sol#L100))

The `KUMASwap::initialize` function casts the result of `term / 30 days` to a `uint16` variable unsafely. We advise the casting operation to be performed safely by ensuring that the `term / 30 days` calculation's result does not exceed the maximum value that a `uint16` variable can hold (i.e. `type(uint16).max`). To note, the built-in safe arithmetic of Solidity **does not protect against casting overflows**.

## `Blacklist.sol`

### BLT-01L: Inexistent Sanitization of State Transitions (Affected Lines: [L38](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/Blacklist.sol#L38), [L47](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/Blacklist.sol#L47))

The `Blacklist::blacklist` and `Blacklist::unBlacklist` functions do not ensure that the previous state of a `_blacklisted[account]` was the opposite of what it is being set to. We advise this sanitization to be applied to prevent misuse of the functions as well as misleading `Blacklisted` / `UnBlacklisted` events.

## `KYCToken.sol`

### KYC-01L: Weak Definition of Owner (Affected Lines: [L49](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/KYCToken.sol#L49))

The `KYCToken::mint` function accepts a `kycData` argument that is meant to contain data about a KYC'd person. The `kycData` contains an `owner` argument that does not appear to be sanitized, permitting KYC data to be minted to an arbitrary `to` address when the `owner` of the `kycData` may be someone else. We advise the `mint` function to permit minting of the `kycData` by either enforcing `to` to equal the data's `owner` member or by not accepting a `to` argument altogether, minting the `kycData` directly to its `owner`.

## `MCAGAggregator.sol`

### MAR-01L: Inexistent Sanitization of Maximum Answer (Affected Lines: [L69-L72](https://github.com/code-423n4/2023-02-kuma/blob/main/src/mcag-contracts/MCAGAggregator.sol#L69-L72)) 

The `MCAGAggregator::setMaxAnswer` function does not apply any sanitization on the input `newMaxAnswer`, permitting even negative values to be set. We advise the function to mandate at least a non-zero `newMaxAnswer`, applying the same check in the `constructor` of the contract.

## `WadRayMath.sol`

### WRM-01L: Incorrect Code Merge (Affected Lines: [L127-L137](https://github.com/code-423n4/2023-02-kuma/blob/main/src/kuma-protocol/libraries/WadRayMath.sol#L127-L137))

The referenced `WadRayMath::rayPow` implementation is not present in the Aave V3 codebase and is instead merged from the `WadRayMath` implementation of Aave V1. The Aave V1 implementation is compiled with a `pragma` statement of `^0.5.0`, rendering the statements within `rayPow` performed using unchecked arithmetics in the original implementation.

We advise the `rayPow` function's body to be wrapped in an `unchecked` code block, replicating the original behavior of `WadRayMath` in Aave V1. We would like to note that while presently not a vulnerability, code copied from other projects should execute in the same format it originates from.